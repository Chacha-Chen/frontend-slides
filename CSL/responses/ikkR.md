# ikkR

We thank the reviewer for the positive comments about our work. In particular, we also find the CSL-next approach interesting because of its reduced overhead and since it does not use any prompt but also performs better than SL/SL(norm). We also appreciate the helpful critical feedback. We attempt to address each of the points that the reviewer raise in what follows: 


## Hallucination Detection
We thank the reviewer for the suggestion.
Uncertainty quantification of confidence scoring are indeed highly related to hallucination detection.
While we typically do not refer to our research as hallucination detection, we did find that many papers frame hallucination detection to be the same as our experiment on AUROC (e.g. Snyder et al. 2023, Kossen et al. 2024).
It is indeed our intention to continue investigating the problem of hallucination in the future, and we would greatly appreciate perspectives on how to evaluate hallucination in (closed-book) generation apart from predicting whether the answer is correct/incorrect. 



References used in this answer:
* Snyder, Ben, Marius Moisescu, and Muhammad Bilal Zafar. "On early detection of hallucinations in factual question answering." arXiv preprint arXiv:2312.14183 (2023).
* Kossen, Jannik, et al. "Semantic Entropy Probes: Robust and Cheap Hallucination Detection in LLMs." arXiv preprint arXiv:2406.15927 (2024).


## Related Works
We than the reviewer for the suggestions. We will include them in the related works section in the next version of the paper. 

## Factuality Inocrrect Cases
We did try to see the case where LLM is factually incorrect.
However, typically the answers are all over the place. 
Thus, it is very difficult to decide what are actually important tokens (as they are just not related to the reference answers at all), or very trivially good (e.g. the reference is "No" but the answer is "Yes"). 
On the other hand, we somewhat believe that confidence itself from the model is not necessarily a score for factuality (even if we ignore the "lexical component"), as the model could be over-confident or may be making educated guesses.
We think of CSL as reducing the noise in sequence likelihood to better represent the model's own confidence, but cannot completely change the game if the model has a very bad underlying confidence about its own answer (e.g. generating nonsense tokens with high probabilities consistently).
However, we think that the approach may be connected more tightly to factuality by tuning the LM on a gold-standard dataset in a specific domain of use.


## Case Study in Figure 5
We are sorry, but we don't fully follow the reviewer's concern about examples in Figure 5, as we think these tokens are mostly important tokens given the context.
If the reviewer can clarify further, we will be happy to provide answers. 
For now, we also hope the following discussion helps.
In Figure 5, we were also trying to highlight that that matching sub-word tokens to human intuition can be tricky.
Sometimes looking at subwords tokens lead to slightly different impressions than whole words: For example, although "guardian" as a whole receives -6\% weight, it's "_guard" (-7\%) and "ian" (+1.3\%).
(In comparison, we have "_he" (+6\%), "_not" (+2.6\%), "_fit" (+13.1\%).)
Due to the soft nature of the attention mechanism (and the intrinsic difficulty to label "important" tokens by hand), we discussed in Limitations that humans might not always agree with CSL on the important tokens.
The attention for CSL-Next is generally similar (likely due to the step on picking heads), but we feel it's slightly less intuitive than CSL in our spot checks.
We do hope to further improve the current method for more interpretability, which could, for example, use fine-tuning.