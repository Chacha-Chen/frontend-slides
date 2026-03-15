# KWCw

We thank the reviewer for the constructive feedback. 
To begin with, we would like to emphasize that this paper is intended as a first step towards UQ for black-box LLMs, which is arguably an improvement over the existing white-box UQ methods. 
Thus, we largely follow the experiment settings in [23], but with improvements in terms of evaluation metrics and more diverse LLMs. 

Below we present responses to each of the points raised:

### Sampling Parameters

Thank you for the suggestion on exploring more sampling parameters. 
Due to the limited computational resources, we opted for experimenting with more recent LLMs/datasets instead of experimenting with more sampling parameters.
This is also due to the fact that we are not reading the logits, so temperature plays a somewhat less important role than [23].
To some extent, we want to keep this sampling process ``black-box`` as well: 
For the baseline (Semantic Entropy) we used the code provided by the original author and its default parameters. 
For our experiments, we set temerature=1 and `top_p=1` (same as OpenAI's API default parameters).
We are running experiments using temperature=0.5, but due the API limit we have not yet finished the experiments. If the paper is accepted, we can add such experiments in the appendix.
On the partial results we are observing similar conclusions, however, the white-box methods improved a little bit.
We do note that in the extreme case where temperature=0, our methods stop working, but some white-box method will still work ([23] still requires sampling though).
We will add this discussion to our limitations (as the reviewer mentioned in the limitation section).



%Since this method in this work requires sampling multiple answers from an LLM, and this work also suggests that their method can be used to find the best answer from multiple samples (similar to self-consistency prompting), it would make sense to also evaluate the confidence estimates based on the best answer available. Since QA systems, in practice, should provide the best available answer and this method already samples multiple answers to estimate confidence, the strongest setting to evaluate AUROC would be using the best answer and its confidence estimate.
## AUROC with the best answer

The reviewer suggests a third task which combines the first (UQ only, like in [23]) and the second (picking the best answer).
We think this is a great suggestion.
The reason why we didn't have this setting is because only $P(true)$ provides a confidence measure among the baselines, and Table 4 already shows the usefulness of confidence measure in improving accuracy without rejection.
(Recall that Table 1-4 are intended to evaluate the quality of $U$ and $C$).
Please refer to Table 1 in the additional page (pdf) for results on such experiments. 


%While the authors discuss the differences between uncertainty/confidence and epistemic/aleatoric uncertainty and present methods for estimating each, it's unclear whether their methods based on lexical similarity or answers or NLI systems can handle their motivating example (a single question that has multiple correct answers). Further analysis is required to establish whether their method appropriately handles these types of inputs.
## Single Question with Multiple Correct Answer
The NLI-based methods are *not* solely "based on lexical similarity", but could capture simple semantic similarity as well.
However, the reviewer mentions a very interesting question that we also discussed in the main text (from L316) and Appendix （from L596). 
We argue that the challenge here lies mostly in the evaluation.
Compared with rougeL in [23], our GPT-based evaluation already identifies many such cases (e.g. L602) where lexically different answers mean the same thing, but to properly evaluate the quality of the generations on open-ended questions, one might need to hire many educated full-time employees like OpenAI did. 
This is beyond the ability of most labs including ours, and is probably why [23] chose to use a simple rougeL metric on the QA question to begin with.
We definitely hope future research, especially those conducted in the industry with ample resources, could explore (and publish) the more challenging task on highly open-ended questions.




### Questions
> What is the NLI model used in this work? What was it trained on?

We followed [23] and used the same deberta-large model (See L158).


> What prompt is used in the P(true) baseline?  ... some of which also utilize multiple sampled answers from the QA model.

Instead of sampling multiple times (which is quite expensive), we peek into the logit and take softmax directly. 

> The performance of the Semantic Entropy baseline seems to be significantly lower than the performance reported in the original work  under identical settings (TriviaQA w/ LLAMA).

In [23], they used only OPT but no LLAMA. 
In our replication of the OPT experiments (Table 3), we actually record *higher* AUROC for Trivia (86 vs 81) and slightly lower for coqa (72 vs 76). 
One difference is related to the use of gpt-3.5 as the evaluator of answers (L235-241). 
We found that rougeL is worse in terms of evaluating the quality of the answers.
Since whether the answer is correct changes, the AUROC changes as well. 
Note we use the official implementation of [23] to replicate their experiment. 

### Limitation: Sampling
We agree this is a limitation (which is probably more likely to overcome in a white-box setting than a black-box one).
We will add this to the main paper near other limitations in the conclusion section.