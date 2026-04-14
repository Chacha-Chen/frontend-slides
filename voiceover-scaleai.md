# Voiceover Script — Scale AI Talk (~20 min)

---

## Slide 1 — Title (~20 sec)

Hi everyone, thanks for having me. My name is Chacha Chen. Today I want to talk about why getting the right answer is not enough — and how we can measure and improve how LLMs actually reason.

---

## Slide 2 — About Me (~1.5 min)

A quick intro. I'm currently at Apple AIML, where I work on LLM reasoning and post-training data and evaluation.

*(click)* Before that, I did my PhD in Computer Science at the University of Chicago, where I focused on AI safety, human-AI interaction, scalable oversight, and uncertainty quantification.

*(click)* I also interned at Microsoft Research, Amazon AWS, and IQVIA — where I worked on evaluating and improving LLM capabilities.

---

## Slide 3 — Research Overview (~2 min)

Here's an overview of my research. It falls into three threads.

The first is about evaluating reasoning quality. For example, at Apple, I proposed a new metric called Zellner's Delta to measure whether LLMs update their beliefs correctly. I also worked on evaluating GPT-4V for radiology — we found that fluent outputs don't mean clinically correct outputs. And CLEAR is a framework for clinically grounded evaluation of radiology reports.

The second thread is about human-AI collaboration and oversight. I've studied whether machine explanations actually help humans make better decisions. I also worked on AI-assisted medical diagnosis, and more recently on scalable oversight through collaborative disagreement resolution.

The third thread is about building robust systems — from uncertainty-aware health models, to RL post-training with safety-gated reward design, to synthetic data pipeline engineering.

---

## Slide 4 — Today's Focus (~20 sec)

Today I'll go deep into one research project — "Are LLMs Optimal Information Processors?" — which is about measuring how well LLMs reason step by step.

---

## Slide 5 — What Current Evaluations Miss (~1.5 min)

So here's the problem. In current benchmarks, we give the model all the evidence at once, it gives an answer, and we check if it's right or wrong. That's it.

*(click)* But in the real world, evidence comes in pieces. A doctor doesn't get all the test results at the same time. It's important to have the right confidence at every step — not just at the end.

*(click)* What's hidden from current evaluation is this whole middle part — how does the model update its beliefs as each new piece of evidence comes in?

---

## Slide 6 — Three Doctors (~1 min)

Let me give you a simple example. A patient has chest pain. Three doctors get lab results one at a time.

The first doctor over-updates — hears "chest pain" and immediately says "heart attack, 95%." No tests needed.

The second doctor under-updates — sees all the results but still says "maybe heartburn, 50%."

The optimal doctor updates step by step, proportional to the evidence. That's what we want.

---

## Slide 7 — Interactive Example (~1.5 min)

Here's a more detailed example. We have four possible diagnoses. The optimal reasoner updates smoothly — each step, the probability changes proportionally to how strong the evidence is.

The flawed model jumps around — it over-commits early, then has to reverse later. Both get the right answer in the end. But the paths are very different. The flawed model would make bad decisions if you stopped it in the middle.

---

## Slide 8 — Evals Only See the End (~1 min)

This is the key problem. Current evaluations only look at the final answer. Both models get marked "correct." But one reasoned well and the other didn't. We introduce Zellner's Delta to measure this difference — at the bottom you can see the optimal path has Delta close to zero, the flawed path has large spikes.

---

## Slide 9 — The Metric (~1.5 min)

So what is Zellner's Delta? It's simple. Delta equals information out minus information in.

Information out is how much the model's prediction changed — measured by KL divergence.

Information in is how strong the evidence was — measured by expected log-likelihood ratio.

When Delta equals zero, the model did exactly what Bayes' rule says. When Delta is larger than zero, the model deviated from optimal. The bigger the Delta, the worse the reasoning.

---

## Slide 10 — Worked Example (SKIP or skim in 30 sec)

I'll skip the detailed math, but the idea is — if a Bayesian model sees elevated troponin, it should shift from 60-40 to roughly 14-86. That gives Delta equals zero. A model that jumps to 2-98 overshoots — Delta is 0.09.

---

## Slide 11 — Key Distinction (30 sec)

Just to be clear how this is different from existing metrics: accuracy checks the endpoint, calibration checks aggregate confidence, and our Delta checks the reasoning path at each step. That's our contribution.

---

## Slide 12-13 — Experimental Setup & Scale (~1 min)

We ran large-scale experiments — over 10 model variants, both open-source and closed-source, over a million API calls, around 960 million tokens, on 120 A100 GPUs.

---

## Slide 14 — Roadmap (~20 sec)

Here's the story of our results. We measure, diagnose, show consequences, prove it's fixable, and then train for it.

---

## Slide 15 — Results Table (~1.5 min)

Here are the main results. Look at this table. GPT-4o-mini has Delta of 0.18. GPT-5.1, 0.15. Open-source models like Qwen and Ministral range from 0.13 to 0.27.

The key finding: models with similar accuracy can have very different Delta. Accuracy alone doesn't tell you which model reasons better.

---

## Slide 16 — Consequences (~1 min)

Why does this matter? These scatter plots show: when Delta is higher, accuracy drops — especially when you have to commit to a decision early, before seeing all the evidence. This is exactly what happens in real-world agent settings.

---

## Slide 17 — Failure Patterns (30 sec)

We found three types of failures: over-updating, under-updating, and reversed updating — where the model moves in the wrong direction. All of these are invisible if you only check the final answer.

---

## Slide 18 — Can LLMs Be Optimal? (~1 min)

But there's good news. We created a synthetic task called BumbleGrumble where all the probabilities are given in the prompt. GPT-5.1 achieves near-perfect Delta of 0.068. So LLMs can do Bayesian reasoning — the gap in real tasks comes from knowledge uncertainty, not math ability.

---

## Slide 19-20 — Training & Results (~1.5 min)

This means we can fix it. We sample traces from a strong model, rank them by Delta, and fine-tune on the best traces. Standard pipeline, no special tricks.

The results: on synthetic tasks, Delta drops from 0.18 to 0.01 and accuracy jumps from 73% to 98%. On real-world benchmarks like GPQA and MediQ, we also see consistent improvement in both Delta and accuracy.

---

## Slide 21 — Methods (SKIP)

*(Skip this slide — too detailed for 20 min)*

---

## Slide 22 — Takeaway (~30 sec)

The takeaway: accuracy measures where you end up. Delta measures how you got there. For LLMs to be trustworthy, we need both.

---

*(Skip Part 2 slides 23-29)*

---

## Slide 30 — Closing (~1 min)

Looking forward — I think there are three big open questions. First, how do we build evaluation infrastructure that measures the reasoning process at scale? Second, how do we handle error compounding in multi-step agents? And third, how do we maintain meaningful human oversight as agents become more autonomous?

Thank you! I'd love to hear about the challenges you're seeing at Scale and discuss how these ideas might connect.
