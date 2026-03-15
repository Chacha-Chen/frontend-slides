# LJF2

We are grateful to the reviewer for the constructive comments.
To begin, we would like to emphasize that this paper focuses on the uncertainty quantification of the LLM, and the experiment setups largely follow the white-box baseline (Kuhn et al. 2023).
We have made revisions to our manuscript as well as performed additional experiments following the reviewer's suggestions. 
Below we present responses to each of the points raised:

## Clarification of Details.

> what DeBERTa model used for NLI? Was it trained using NLI data?

For a fair comparison, we followed (Kuhn et al. 2023) and used the same official pretrained DeBERTa-large on hugginface, which was fine-tuned on MNLI data.
More training details of this model could be found on the model card of `deberta-large-mnli` on huggingface.



## Uncertainty vs Confidence

We apologize for the confusion here.
The structure was that 3.1 defines uncertainty and 3.2 defines confidence (which is different).
$U$ is a property of the model's perceived posterior and depends only on $x$ (see L3-5 in Section 3.1, and Eq 1).
$C$ depends on both $x$ and the answer $\mathbf{s}$ (L2-L3 of Section 3.2 and Eq 2).
These are commonly used terms in two somewhat disconnected domains and we have made updates to Section 3.2 to emphasize this distinction.

## Recommended Variation / Why a particular method outperforms

Thank you for the constructive question/suggestion.
In section 5.3, under *Uncertainty Measures*, our conclusion was that the similarity measure (Section 4.1) seems more important than variants proposed in Section 4.2.
We hypothesize that "entail" works better because generally the output space is huge, and for two generations, "not contradicting" is likely not a sufficient indicator of low-uncertainty (as they can both be bad random responses).
We have updated section 5.3 and Appendix B.7 with examples to reflect this hypothesis.

## GPT evaluation

We would like to clarify that the number of evaluations is *99* per dataset instead of 33 (33 per (model, dataset) pair  * 3 models)
The number of total human evaluations is chosen basing on (Kuhn et al. 2023), which evaluated 200 answers for two datasets (we have 297 on three).
We tried our best to verify the reliability of GPT evaluation, but the verification process is actually quite slow, as for CoQA we need to read the passage first before judging the answers, and even for TriviaQA or NQ, sometimes seemingly unrelated answers are just two names of the same person and require careful research.
We agree that a larger-scale human evaluation with mulitple annotators is more desirable, but the main focus of this paper is UQ, and we think the topic of automatic evaluation is an important research area by itself.
Existing UQ literature almost always use purely lexical measures like ROUGE-based automatic evaluation, and we believe the GPT evaluation is better (please refer to Appendix B.3 for a discussion).


To address your concern, we increased the number of annotations to 200 per dataset (50 per (model, dataset) * 4 models) and the new accuracies are 89.4/96.5/93 for coqa/trivia/nq.
Using the same annotations, the ROUGE-L based metric's accuracy is 81.5/93.5/85. 
If we construct a confidence interval we could reject that "GPT and ROUGE are equally good" (our estimate is that further shrinking the confidence interval by half requires about 20 hours of annotation time).
We found that llama2 (newly added) tends to generate different expressions of the same answer, which is sometimes judged incorrect by GPT.
Excluding llama2, the GPT accuracies are 90.6/98/94 for coqa/trivia/nq.



## Other Questions


> What is the reasoning for a_{NLI, contra}? If you use a standard NLI model which has three classes, Eq 4 (right) would take into account the entailment and neutral class (since it is 1-p_{contra}, is that correct?

$a_{NLI, contra}$ accounts for both neutral and entailment. 
We could consider the difference between $a_{NLI, contra}$ and $a_{NLI, entail}$ as whether we think "neutral" is closer to “similar” or “dissimilar”.

> Unclear why uncertainty is used for expected accuracy and model confidence for individual accuracy.

The setting of uncertainty + individual accuracy was in Table 11 (now 10), and we discuss in Appendix C.2 why we choose not to include it in the main text. 
Essentially, since $U$ does not depend on a particular generation (i.e. it is a property of the model's posterior, for a random generation), we think it should be used to predict the (estimated) expected accuracy (which also depends on the posterior/a random generation).

> Provide examples on when the uncertainty and confidence estimation aligns/doesn't align with the prediction

We've added more examples (e.g. Fig 5-7) in the Appendix in the updated pdf, but we are not entirely sure with what you mean by aligning with the prediction.
Could you check if the new examples address your concerns, or clarify your suggestion? Thank you!
