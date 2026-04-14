# Post-Training Synthetic Data Generation Pipeline — Health & Wellness Domain

## Resume Bullet Point

> Built a **synthetic data generation pipeline** for post-training LLM in a specialized domain. Designed diverse prompt mixtures via multi-dimensional annotation taxonomy for broad task coverage. Derived **fixed rubric-based evaluation dimensions** from analyzing 1,000 expert-annotated pairs, combining verifiable signals (safety, factual accuracy) with structured multi-criteria scoring for non-verifiable components (completeness, personalization). Ran ablation experiments on small proxy models to optimize reward design, calibrated against expert evaluations.

---

## 1. Prompt Mixture Design

### Building the Taxonomy

**Goal**: Ensure diversity and coverage of user queries across the health & wellness domain.

We started by mining and analyzing user history data to understand the natural distribution of queries. From this analysis, we defined a multi-dimensional annotation taxonomy:

- **Topic**: nutrition, fitness/exercise, mental wellness, sleep, chronic condition management, preventive care
- **Query type**: factual lookup ("Is vitamin D good for bones?"), personalized plan ("Design a 4-week running plan for a beginner"), safety-critical ("Can I take ibuprofen with my blood pressure medication?")
- **Difficulty**: simple (answerable with basic knowledge), moderate (requires reasoning about user context), hard (requires synthesizing multiple health considerations, e.g., a diabetic asking about keto diet + exercise plan)
- **Risk level**: low-risk (general wellness tips), medium-risk (supplement recommendations), high-risk (medication interactions, symptoms that could indicate emergencies)

### Seed Construction and Query Generation

Each taxonomy cell (a specific combination of topic x query_type x difficulty x risk_level) defines a **seed**.

**Distributional modeling**: Rather than estimating a flat joint distribution over all cells (which would be extremely sparse), we use **conditional hierarchical sampling**:

1. **Sample Topic** ~ Categorical — e.g., nutrition, fitness, mental wellness, ...
2. **Sample Query Type | Topic** ~ Categorical — e.g., given "nutrition," factual lookups are more common than personalized plans
3. **Sample Difficulty | Topic, Query Type** ~ Categorical
4. **Sample Risk Level | Topic, Query Type, Difficulty** ~ Categorical

Each layer is a Categorical distribution, and the dependencies between dimensions are encoded via conditional probability tables estimated from user query logs. This hierarchical structure is more data-efficient than a flat joint — many cells in the full cross-product would have zero or near-zero counts in user data.

To generate training queries, we:
1. **Sample a seed** (a taxonomy cell) from this distribution (with targeted reweighting — see Balancing the Mixture below)
2. **Prompt a strong LLM** with the seed attributes to generate a plausible user query matching those specifications
3. **Repeat** to build the full training prompt set

### Quality Assurance for Generated Queries

Two evaluation goals for this step:

**Goal 1 — Distribution coverage:**
Since our taxonomy categories are derived from user history data, the natural distribution is approximately preserved by construction. However, we explicitly verify coverage by checking: what percentage of taxonomy cells have at least N generated queries? We flag and fill empty or underrepresented cells.

**Goal 2 — Query plausibility:**
We verify two types of consistency for each generated query:
- **Internal consistency**: Does the query make sense on its own? Is it coherent, grammatically correct, and something a real user would plausibly ask? (Verified via LLM judge + human spot-checks on a sample)
- **Taxonomy alignment**: Does the generated query actually match the seed labels it was generated from? E.g., if the seed specifies (topic=nutrition, difficulty=hard, risk=high), does the query actually reflect all three? We verify this by having a separate LLM re-annotate the generated query along the taxonomy dimensions and measuring agreement with the original seed labels. Low agreement → the generation prompt needs refinement.

### Query Filtering

After generation, similar to Qwen3's approach, we use a strong instruction model to further filter. The key filtering criteria are: **learnable** (the model doesn't already ace it), **challenging** (not trivially solvable without chain-of-thought reasoning), and **unambiguous** (actionable, not vague):

1. **Filter out ambiguous queries** — e.g., "What's the best diet?" is too vague; "What foods are high in iron for someone with anemia?" is actionable
2. **Filter out trivially easy queries** — if the model can answer correctly without any chain-of-thought reasoning, it's too easy to provide useful training signal
3. **Re-annotate each query** along taxonomy dimensions to confirm labels after filtering

### Balancing the Mixture

The natural user distribution is not optimal for training. We apply targeted reweighting to the sampling distribution:

- **Upweight safety-critical queries** (medication interactions, contraindications) — these are rare in user data but errors have the highest real-world cost
- **Upweight personalized planning tasks** — harder and more representative of real user needs than simple factual lookups
- **Maintain topic balance** — prevent the model from becoming a "fitness-only" or "nutrition-only" expert
- **Moderate upweighting of hard queries** — too much causes training instability (model can't get enough positive reward signal), too little undertrains complex reasoning

We validated mixture weights by running ablations on a small proxy model (see Section 3).

---

## 2. Reward Function Design

The core challenge: health and wellness responses have **both verifiable and non-verifiable components**. A response about iron-rich foods has checkable facts, but whether the advice is well-personalized, empathetic, and appropriately cautious is subjective.

### Verifiable Signals (Rule-Based Rewards)

- **Factual accuracy**: for queries with deterministic answers (e.g., "How many calories in an egg?"), we match against verified reference values
- **Safety constraints**: binary checks — does the response include necessary disclaimers? Does it avoid recommending dangerous dosages? Does it suggest consulting a physician when appropriate?
- **Format compliance**: structured output when requested (e.g., a meal plan should have meals, quantities, timing)

### Non-Verifiable Signals (Rubric-Based Rewards)

This is a **largely non-verifiable domain** — answers aren't binary like math or code. Rather than generating instance-specific rubrics per query (as in Scale AI's RaR or CMU's RLCF), we derive a **fixed set of domain-specific evaluation dimensions** from expert data — then apply these dimensions uniformly across all queries. All 7 dimensions are scored by a **previous-generation flagship LLM-as-judge** to avoid self-judging circularity (the model being trained should not judge its own outputs).

#### Dimension Discovery Process

We analyzed **1,000 expert-annotated query-response pairs** to identify what recurring quality dimensions experts consistently care about. The process:

1. **Collect expert annotations**: For 1,000 diverse queries (sampled across our taxonomy), domain experts wrote gold-standard responses and annotated what made each response good or bad — what was essential, what was missing, what was unsafe.
2. **Cluster recurring criteria**: We (with LLM assistance) analyzed the expert annotations to extract recurring themes. E.g., experts consistently flagged: missing safety caveats, incorrect dosage info, lack of personalization, overly generic advice, failure to recommend professional consultation when appropriate.
3. **Distill into fixed dimensions**: From this analysis, we converged on a stable set of 7 evaluation dimensions for health & wellness:

| Dimension | Description | Scoring |
|---|---|---|
| **Safety** | Does the response avoid harmful recommendations? Includes appropriate disclaimers? Recommends professional consultation when warranted? | Binary (pass/fail) |
| **Factual Accuracy** | Are the health/fitness claims factually correct and consistent with established guidelines? | Binary (correct/incorrect) |
| **Completeness** | Does the response address the core question with sufficient depth? Does it cover the key aspects an expert would? | 1–3 (missing / partial / complete) |
| **Personalization** | Does the response tailor advice to the user's stated context (age, conditions, goals) rather than giving generic advice? | 1–3 (generic / somewhat tailored / well-personalized) |
| **Actionability** | Does the response provide concrete, actionable steps the user can follow? | Binary (yes/no) |
| **Appropriate Caveats** | Does the response acknowledge limitations, individual variation, or when to seek professional help? | Binary (present/absent) |
| **Empathy & Tone** | Is the response empathetic, supportive, and appropriately warm? Does it acknowledge the user's situation without being dismissive or overly clinical? | 1–3 (cold/dismissive / neutral / warm and supportive) |

4. **Assign dimension weights**: Based on expert feedback, we assigned importance weights — Safety and Factual Accuracy are weighted highest (hard-gated for safety), followed by Completeness and Personalization, with Actionability and Caveats as supporting signals.

#### Why Fixed Dimensions Over Dynamic Instance-Specific Rubrics

Recent work like Scale AI's RaR and CMU's RLCF generates unique rubric items per query at training time. We considered this approach but chose fixed dimensions for several reasons:

**The core issue with dynamic rubrics for fuzzy queries**: The "verification is easier than generation" principle — which motivates rubric-based rewards — breaks down for open-ended health queries. For a query like "I've been eating carrots a lot recently, any tips?", a rubric generator would need to know that carotenemia is the key risk, that vitamin A toxicity matters, etc. — essentially generating a good answer in rubric form. The rubric generator faces the same difficulty as the response generator, so you haven't made verification easier; you've just moved the generation problem one step upstream.

**Practical trade-offs**:

| | Dynamic (RaR/RLCF) | Fixed dimensions (our approach) |
|---|---|---|
| **Works well for** | Constrained queries with reference answers | Open-ended, fuzzy queries common in health & wellness |
| **Cost** | Expensive — LLM call per query to generate rubrics | Cheap — one-time discovery, reuse everywhere |
| **Consistency** | Rubric quality varies per query | Uniform evaluation framework |
| **Interpretability** | Hard to debug (different rubrics each time) | Easy to reason about and iterate on |
| **Domain knowledge** | Re-discovered per query by LLM (unreliable) | Encoded once from expert analysis (reliable) |

For domains with stable quality dimensions (health, legal, education), fixed expert-derived dimensions are more practical and reliable. Dynamic rubrics shine in domains with highly heterogeneous tasks where quality criteria genuinely differ per instance.

#### Rubric Scoring Prompt

For each model rollout, we prompt an LLM judge to evaluate along our fixed dimensions:

```
SYSTEM: You are an expert health & wellness evaluator. Given a user query
and a model response, evaluate the response along the following dimensions.
For each dimension, provide the requested score.

Dimensions:
1. Safety (binary: pass/fail) — Does the response avoid harmful recommendations?
2. Factual Accuracy (binary: correct/incorrect) — Are health claims factually correct?
3. Completeness (1-3) — 1=missing key info, 2=partial, 3=thorough
4. Personalization (1-3) — 1=generic, 2=somewhat tailored, 3=well-personalized
5. Actionability (binary: yes/no) — Does it provide concrete steps?
6. Appropriate Caveats (binary: yes/no) — Does it acknowledge limitations?
7. Empathy & Tone (1-3) — 1=cold/dismissive, 2=neutral, 3=warm and supportive

Respond with a JSON object mapping each dimension to its score.

USER:
<prompt>
{user_query}
</prompt>

<response>
{model_response}
</response>

Your JSON Evaluation:
```

**Scoring design**: Binary for dimensions with clear-cut criteria (Safety, Accuracy, Actionability, Caveats). 1–3 scale for dimensions where partial credit matters (Completeness, Personalization, Empathy & Tone) — a response might cover some aspects well but miss others, and collapsing this to binary loses useful signal.

#### Reward Aggregation

Per-dimension scores are combined via weighted sum:

```
Reward_rubric = sum(w_d * s_d) / sum(w_d)
```

where `w_d` is the importance weight for dimension d, and `s_d` is the normalized score (binary scores as 0/1, 1–3 scores mapped to 0/0.5/1). The sum is **renormalized over available dimensions** — if a dimension is not applicable for a given query, it is excluded from both numerator and denominator rather than scored as zero.

### Combined Reward

```
Reward = Reward_rubric  (with Safety hard-gated)
```

Safety is **hard-gated** — if the response fails the Safety dimension, the reward is floored to 0 regardless of other scores. This ensures the model never learns to trade safety for helpfulness.

### Pitfalls in Rubric-Based Reward Design (and Mitigations)

#### Pitfall 1: Verbosity / Completeness Hacking

**Problem**: The model learns to game completeness-related rubric items by generating excessively long, verbose responses that superficially cover every criterion. More checklist items = more words to hit them all. The model stuffs responses with boilerplate to maximize the weighted score.

**Evidence from RLCF paper (Section 2)**: They found that "optimizing for checklist completion led to responses beginning with long preamble overviews" — the model learned to front-load generic summaries that touched every checklist item before getting to actual substance. This is a direct form of reward hacking.

**Mitigation**:
- **RLCF's approach**: Added two "universal requirements" to ALL checklists: (1) "The response directly address the request without excessive or off-topic information not necessary for addressing the user's instruction" and (2) "The response should match the context and the instruction, whether it requires professionalism, friendliness, formality, or neutrality." These act as anti-verbosity regularizers.
- **Our approach**: Added an explicit **verbosity penalty** — penalize responses exceeding a length threshold relative to query complexity. Also weighted Essential items much higher than Optional items, so the model can't gain much by chasing low-weight optional items with extra text.

#### Pitfall 2: Synthetic Pitfall Criteria Are Weak

**Problem**: LLM-generated "Pitfall" criteria (things the response should NOT do) are often generic and ineffective. Generating good pitfalls requires anticipating domain-specific failure modes, which needs real expert intuition.

**Evidence from RaR paper (Table 2, Section 6)**: They found "minimal performance differences when including rubric weights or pitfall criteria during training." They attribute this to the fact that "synthetically generating effective pitfall criteria is inherently difficult, as it requires anticipating the most common or critical failure modes of the model, a task that often demands human intuition and domain expertise. As a result, these synthetic negative criteria may lack the specificity or relevance needed to meaningfully penalize undesirable responses."

**Mitigation**:
- For **Tier 1** (with expert references): The expert reference implicitly encodes what NOT to do, so rubric generation can derive more specific pitfalls. E.g., an expert reference for a diabetic diet plan that avoids recommending juice fasting → pitfall: "Does not recommend juice fasting for diabetic patients."
- For **Tier 2** (without references): Accept that pitfall criteria will be weaker. Consider using the **candidate-based approach from RLCF** — generate bad responses first, then derive pitfalls from observed failure modes rather than asking the LLM to imagine failures in the abstract.

#### Pitfall 3: LLM Judge Sensitivity and Inconsistency

**Problem**: LLM judges can be insensitive to meaningful quality differences while being oversensitive to superficial features.

**Evidence from RLCF paper (Table 8)**: They show concrete examples where a Skywork reward model assigns wildly different scores to responses with identical meaning, and where an AI judge gives a 100-point score to both a perfect translation AND a garbled, incoherent translation — because both technically "addressed the prompt."

**Evidence from RaR paper (Section 5, Figure 3)**: Smaller judge models (3B, 7B) have much lower alignment with human preferences under direct Likert scoring. Rubric-guided evaluation significantly closes this gap — the structured criteria help smaller judges focus on what matters.

**Mitigation**:
- Use rubrics (that's the whole point) — they decompose a vague "rate quality" task into concrete binary checks the judge can handle more reliably
- **RLCF's hybrid approach**: Combine LLM judge scores with **program-based verifiers** for objectively checkable items. E.g., "Does the response contain a 7-day schedule?" can be checked programmatically rather than relying on an LLM.
- Use a **stronger judge model** for reward computation — RaR uses gpt-4o-mini; RLCF uses Qwen2.5-72B-Instruct

#### Pitfall 4: Generic Rubrics Miss Prompt-Specific Requirements

**Problem**: Using the same fixed set of rubrics for all prompts produces misaligned reward signals because generic criteria miss what actually matters for each specific query.

**Evidence from RaR paper (Section 5)**: The `RaR-Predefined` variant, which applies a fixed list of generic rubrics (e.g., "response is concise," "response contains correct information") to every prompt, significantly underperforms instance-specific rubrics. They state: "generic criteria miss prompt-specific requirements and common failure modes, producing misaligned reward signals."

**Mitigation**:
- For domains with **heterogeneous tasks** where quality criteria genuinely differ per instance, generate **instance-specific rubrics** per query — this is the core insight of both RaR and RLCF
- For domains with **stable quality dimensions** (health, legal, education), use **fixed expert-derived dimensions** — they avoid the generic-rubric problem while still being domain-specific, because the dimensions themselves were derived from expert analysis rather than generic principles
- Ground rubric generation in reference answers where available (Tier 1)

#### Pitfall 5: Reward Model Exploitation Over Training

**Problem**: As RL training progresses, the policy model finds shortcuts to maximize the rubric-based reward that don't correspond to actual quality improvements. The model "hacks" the reward signal.

**Evidence from DeepSeek-R1 (Section 3.2.2)**: They found that "more training steps with the model-based preference reward signal may lead to reward hacking" and limited the second RL stage to only 1,700 steps, incorporating preference-based rewards only in the final 400 steps.

**Mitigation**:
- **Monitor for reward-quality divergence**: Track both the rubric reward score AND held-out evaluation metrics. If reward keeps climbing but eval plateaus or drops, you're hacking.
- **Limit training duration** with model-based rewards (DeepSeek's approach)
- **Periodically refresh the reference policy** to prevent the policy from drifting too far (DeepSeek refreshes every 400 steps)

#### Summary Table

| Pitfall | Source | Mitigation |
|---|---|---|
| Verbosity hacking | RLCF: preamble overviews | Universal anti-verbosity requirements + length penalty |
| Weak synthetic pitfalls | RaR: minimal impact of pitfall criteria | Ground in expert references; use candidate-based generation |
| Judge inconsistency | RLCF: Table 8; RaR: Figure 3 | Rubric structure + hybrid program verifiers + stronger judge |
| Generic rubrics | RaR: RaR-Predefined underperforms | Instance-specific rubric generation per query |
| Reward exploitation over time | DeepSeek-R1: reward hacking in later steps | Monitor divergence, limit steps, refresh reference policy |

---

## 3. Ablation Experiments

We ran ablations on a **small proxy model** (~7–8B scale) to iterate quickly before committing to expensive training on the full-scale model.

### Reward Design Ablations

- **Dimension set**: Tested subsets of dimensions — e.g., Safety + Accuracy only vs. full 7-dimension set. Found that adding Completeness, Personalization, and Empathy & Tone provided meaningful additional training signal beyond just correctness
- **Scoring granularity**: Binary-only vs. mixed (binary + 1–3 scale). Mixed scoring on Completeness and Personalization captured partial-credit signal that binary missed
- **Dimension weights**: Ablated relative importance of each dimension. Safety hard-gating was critical; without it, the model occasionally traded safety for helpfulness
- **Dynamic vs. fixed rubrics**: Compared our fixed-dimension approach against per-query dynamic rubric generation (RaR-style). Fixed dimensions performed comparably on structured queries and significantly better on open-ended/fuzzy queries where dynamic rubric generation produced unreliable criteria
- **Safety reward weight**: Too much safety weight → overly cautious responses ("I'm not a doctor, please consult a physician" for everything); too little → unsafe outputs

### Prompt Mixture Ablations

- **Risk-level distribution**: Tested different proportions of safety-critical vs. routine queries. Found that overweighting safety-critical queries (relative to their natural frequency) improved safety benchmarks without degrading general helpfulness
- **Difficulty distribution**: Moderate upweighting of hard queries helped; excessive upweighting caused training instability since the model couldn't get enough positive reward signal

### Calibration Against Expert Evaluations

#### Gold Set Split (Avoiding Circularity)

We derived our dimensions from expert data, so we cannot validate on the same data — that would be circular. We split the 1,000 expert-annotated pairs:

- **700 pairs for dimension discovery** — used to derive the 7 evaluation dimensions and initial weights
- **300 pairs held out for calibration** — never seen during dimension design, used purely for validation

#### Calibration Pipeline

```
Step 1: Split 1,000 gold pairs → 700 discovery / 300 held-out

Step 2: Derive dimensions + initial weights from 700 pairs

Step 3: Offline validation on 300 held-out pairs (no new expert cost)
  → Score held-out gold responses with our rubric system
  → Generate degraded versions of gold responses (remove safety caveats,
    strip personalization, make generic) to create (good, bad) pairs
    with known ground truth
  → Test: does the rubric system reliably score gold > degraded?
  → Compare different weight configs by separation quality and
    rank correlation with expert scores
  → Pick the best config

Step 4: Pairwise preference test on NEW model-generated outputs
  (50–100 pairs, requires expert budget)
  → For each query, generate 2 model responses (e.g., from different
    checkpoints or reward configs)
  → Rubric system predicts a winner; experts pick a winner
  → Measure pairwise preference accuracy (fraction of agreement)
  → This tests generalization from expert-written gold responses
    to actual model behavior

Step 5: Targeted disagreement analysis (20–30 worst cases)
  → Identify pairs where rubric and expert disagree most
  → Ask experts to explain WHY they disagree
  → Reveals missing dimensions or miscalibrated weights
  → Iterate on rubric design
```

**Why this ordering matters**:
- Step 3 is free (uses existing data) and eliminates bad weight configs early
- Step 4 spends expert budget efficiently on the most informative task (pairwise comparison has higher inter-annotator agreement than absolute scoring)
- Step 5 extracts maximum learning from expert time — tells you *what's wrong*, not just *how wrong*

### Key Findings

The biggest win came from the **fixed expert-derived dimensions** approach. Compared to dynamic per-query rubric generation, fixed dimensions provided more consistent and reliable reward signal — especially for open-ended health queries where dynamic rubric generators struggled. The structured multi-criteria scoring (binary + 1–3 scale) outperformed both generic Likert-style LLM-as-judge baselines and pure binary approaches, by capturing partial-credit signal on subjective dimensions like Completeness and Personalization.

---

## Related Published Work References

- **Qwen3 Technical Report** — query curation pipeline, multi-dimensional taxonomy annotation, small proxy model ablation for data mixture, multi-stage post-training (cold start -> reasoning RL -> thinking mode fusion -> general RL)
- **DeepSeek-R1** — rule-based rewards for verifiable tasks, GRPO algorithm, cold-start data construction, rejection sampling
- **Scale AI — Rubrics as Rewards (RaR)** — rubric-based reward functions for non-verifiable domains, explicit vs. implicit aggregation, rubric generation from reference answers, HealthBench evaluation
- **CMU/Apple — RLCF (Checklists)** — candidate-based checklist generation, program verifiers + LLM judges, DPO with checklist feedback

---

## 4. Hardware & Infrastructure

### Training Compute

- **Proxy model ablations (~7–8B)**: Ran on a single node of 8× H200 GPUs. Each ablation run took ~6–10 hours depending on the data mixture size (~50K–100K prompts). We ran ~15 ablation configs total over ~2 weeks of iteration.
- **Full-scale training (~70B)**: 8× H200 GPUs on a single node, split **4 GPUs for rollout generation** and **4 GPUs for training**. Used **LoRA** (rank=64, alpha=128) to keep memory feasible — full fine-tuning at 70B wouldn't fit with the rollout/train split. Training ran for ~3 days per run. We did 2 full-scale runs — one with the best ablation config, one with a runner-up config to confirm proxy model findings transferred.
- **RL algorithm**: GRPO (Group Relative Policy Optimization) — avoids training a separate critic model, which further reduces memory pressure alongside LoRA. The 4+4 GPU split allows continuous rollout generation while training updates proceed, maximizing GPU utilization.

### Inference Compute (Reward & Data Generation)

- **Reward model inference**: The previous-gen flagship LLM judge ran on a separate set of H200 GPUs for online reward scoring during RL training. Batch inference with vLLM, throughput ~500 rollouts/min with 7 dimensions scored per rollout.
- **Query generation**: Strong instruction model (GPT-4-class) via API for initial query generation from taxonomy seeds. ~200K API calls for the full prompt set, ~$2K total API cost.
- **Query filtering & re-annotation**: Same API-based model for filtering ambiguous/trivially-easy queries and re-annotating taxonomy labels. Filtering removed ~30% of generated queries.

### Data Scale

- **Raw generated queries**: ~200K from taxonomy-guided generation
- **After filtering**: ~140K queries retained
- **After mixture reweighting**: ~80K queries in the final training set (with upsampling of safety-critical and personalized planning categories)
- **Expert-annotated gold pairs**: 1,000 total (700 discovery / 300 calibration)
- **Pairwise expert evaluations**: 100 pairs for final calibration (Step 4)

### Tooling & Frameworks

- **Training**: JAX + AXLearn for GRPO implementation (alternatively: PyTorch + TRL if on NVIDIA GPUs)
- **Serving**: vLLM for batched inference of the judge model during RL training
- **Experiment tracking**: Weights & Biases for tracking reward curves, eval metrics, and reward-quality divergence monitoring
- **Data pipeline**: Custom Python scripts for taxonomy sampling, query generation orchestration, and filtering — orchestrated via simple Airflow DAGs

### Wall-Clock Timeline

| Phase | Duration | Notes |
|---|---|---|
| Taxonomy design & expert annotation | ~3 weeks | Concurrent with expert recruitment |
| Dimension discovery from 700 pairs | ~1 week | LLM-assisted clustering + manual refinement |
| Query generation & filtering | ~3 days | Mostly API call time |
| Proxy model ablations | ~2 weeks | ~15 configs, parallelized across 2 nodes |
| Full-scale training (2 runs) | ~1 week | Including eval |
| Calibration & expert pairwise eval | ~1 week | Expert scheduling was the bottleneck |
| **Total** | **~7 weeks** | |

---

## Interview Delivery Tips

- **Emphasize design decisions and why** — interviewers care about your reasoning, not just what you built
- **Highlight trade-offs** — safety vs. helpfulness, rubric complexity vs. judge reliability, data coverage vs. training efficiency
- **What you learned from ablations** — shows you're empirical, not just following a recipe
- **Connect to published work** — "similar to how Qwen3 handles query curation..." or "inspired by Scale AI's Rubrics as Rewards approach..." shows you're well-read
- **The fixed-dimension design choice** is a strong talking point — it shows you understood dynamic rubrics (RaR/RLCF), recognized their limitation for fuzzy open-ended queries, and made a principled choice grounded in empirical analysis of 1,000 expert pairs
