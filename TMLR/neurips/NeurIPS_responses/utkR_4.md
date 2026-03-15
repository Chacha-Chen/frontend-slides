# utkR

%Weaknesses:
%Details are missing in Section 4.2. How many responses are sampled (so that they can be grouped into semantic sets)?
### W1: How many responses are sampled

This is the parameter $m$ as mentioned in the experiment section (e.g. L282)
In Section 4.2, $m$ is just a notation (much like $N$ in "we have $N$ samples").
A higher $m$ leads to better results (Figure 3, and Appendix) but is more costly.


%The authors assume that we only have the textual output, but not the logits from the neural nets. It’s unclear if “no probability” or “no logits” is a reasonable assumption. Even most of the state-of-the-art large models would give us probabilities.
### W2: The Assumption of "no logits"
We have tried to emphasize that we want to do so for black-box models. In fact, the latest OpenAI API does give us the probability or logits.
This assumption is not so much as a technical assumption on the actual model architecture, but more of a (unfortunate) realistic one where models are increasingly closed-source and served only via API. 

> Would having probabilities gives us a much better uncertainty/confidence metric (compared to the proposed ones)?

In our experiments, the white-box methods actually ([23]) perform worse.
We however conjecture that there should be methods that achieve better results using the probabilities.

%If the authors are experimenting with large LMs, what about the baseline where we just ask the LM how confident they are?
### W3: What about asking the LM directly?
This is an interesting point. 
With enough computation resources, P(true) could be considered such as baseline (by asking the model many time whether it thinks its own answer is correct).
We don't have the computation power to sample, so we directly take the logits and found that it is not performing well.
One could also try prompting the black-box LLM to give a continuous rating (and we performed some ad-hoc experiments manually) of its own answer.
At the time of our experiments, this could only be done on GPT-3.5 (or better) as worse LLMs are not "self-aware" enough, but now it is definitely interesting to conduct experiments on multiple LLMs (Bard, GPT-4, ...), which is a great future research direction.

%In my opinion the title and introduction are a bit deceiving. All the sections before the experiments make me feel like the uncertainty quantification would work on all generation tasks (e.g., machine translation, summarization, story generation, dialogue), but the experiments are only done on question-answering, especially given that the tasks often correspond to very short answer spans (often just one or two words).
### W4: QA task only and short answers
While it is true that we restrict to QA like prior works [31,23,27], the average response contains 5 words and over 10 percent of them are longer than 10 words. 
In fact, the reason why we abandoned rougeL as the evaluaton metric is because we found that the length of the answer itself is too confounded with the evaluation metric.
We do agree this is a limitation shared by recent works using these QA datasets in general, and we'll add this to the limitation discussion.

> So, would the approach work on tasks (1) that are not question answering (2) whose generation length is much longer (say, 20 tokens, or even 50 tokens)?

These are intersting questions to explore in future research.
For (2), in our earlier experiments these methods still work for longer responses, but since no existing work restrict to this subset (which greatly reduces the dataset size and could render the conclusion unreliable), we do not include these in the main paper.
We could add this table to the Appendix.
However, in our opinion, much longer answers typically exist in other tasks, and the main challenge is the automatic evaluation.
Heuristics like rouge have serious limitations.
We are actually working on a different paper focusing solely on the evaluation of longer generations.
%For (1), we restrict to question-answering tasks as suggested in our introduction (we will modify the abstract to emphasize this).


> It’s not sufficiently clear to me why those metrics are chosen (e.g., the reason to choose eccentricity could be better motivated).

Eccentricity is essentially a distance from the cnetroid after we project the responses using graph-based methods.
We will add this motivation to the revision. 

> Additionally, have the authors thought about using metrics that depend on the common BLEU/BLEURT which measure semantic similarity? Are they applicable at all?

We didn't try BLEU-based measures, but included Rouge-based measures in the LexiSim baseline. 


## Minor comments regarding NLI scores
NLI is taken from the recent ICLR spotlight [23] and our paper focuses on extending and improving it in the black-box settings. 
It is one possible way to compute the similarity score, but not necessarily the best.
In fact, even the simple Jaccard is outperforms baselines when used with the $U$ and $C$ measures.


> First, entailment is not symmetric.

Like the reviewer pointed out, this is addressed when we create the $U$ and $C$ measures.
We symmetrize the matrix first (e.g. L195).
We could surely make them symmetric first, in Section 4.1.

> it’s possible that s_j1 and s_j2 are very similar except s_j2 contains one more minor detail

This is definitely a possibility. 
In this case, it is hard to decide whether they are "similar" or not, which is why the question is also fed into the NLI model (see Appendix for the full prompt). 
In general, this "similarity" is the best word we can find for section 4.1, but it does not bear a specific technical meaning per se.

%Questions:
%Line 105: “S represents the random sequence of generated tokens” – do the authors mean that s is a sequence, and S is the set of sequences? How are they “randomly” obtained?
### Q1: Random Sequence S

> do the authors mean that s is a sequence, and S is the set of sequences?

No, we mean that $\mathbf{S}$ is the random variable and $\mathbf{s}$ is a realization. 


>How are they “randomly” obtained?

This is discussed in [23] more, as they take quite a Bayesian perspective.
In that perspective (and Eq.(1)), one could think of this random sequence being generated by the LLM (which can be viewed as a posterior),
In our paper, we assume the LLM is a black-box, so we do not care as much about this formulation.
Eq(1) is only used to exemplify a $U$ measure, and that $U$ typically only depends on the input $x$ (unlike Eq. (2)).