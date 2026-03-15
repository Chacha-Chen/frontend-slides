# GwLD

We thank the reviewer for the appreciation of our approach and pointing out some potential avenues for clarification. Below we address the points the reviewer has raised. 

## Connection between Reweighting and Semantics
The reviewer might appreciate that it is somewhat difficult to evaluate quantitatively how much the semantics are emphasized, before and after the contextualization.
We rely on prompting to capture the context, and then tried to argue, via Figure 2 that the prompt generally increases attention on the important tokens. We also present some case studies in Figure 5 to show that the less important tokens in a correct answer are often down-weighted.

## Temperature Scaling (baselines)
We thank the reviewer for the question about temperature scaling. 
We would like to clarify that temperature scaling is typically used when the confidence is measured in a classification setting (e.g. framing the QA as a multiple choice, like Kadavath et al. 2022).
In free-form NLG, it is unclear how to optimize for the temperature like in Guo et al. 2017 because there is no classification label to tune the temperature.
We'd also like to note that TS in general focuses on calibration of the top class probability, which is not directly related to our research focus (and we continue this discussion in the response to the query about ECE).
If we misunderstood and you are suggesting another way of using temperature scaling, please do let us know.

As for (Tian et al., 2023), we appreciate the reviewer for pointing out this reference and we will of course include it in the related works.
Due to the time constraint during the discussion phase, it difficult to include such experiments right now, and it does not provide prompts for open-book datasets like CoQA.
However, we are running experiments and will try to include it in the next update of the paper.


- Kadavath, Saurav, et al. "Language models (mostly) know what they know." arXiv preprint arXiv:2207.05221 (2022).

## Computational overhead
We do need a small dataset to choose the heads for new language models, but we would like to clarify that we need *one* (not 1024) forward pass to the LLM per question to get all attention values (by setting `output_attentions=True` with hugginface). 
In general, SSL is much cheaper than most baselines that also require a calibration dataset, as the head selection step takes a few seconds (on top of inference on the calibration data), and is even cheaper at inference time. 

## Calibration/Expected Calibration Error (ECE)
We have reliability diagrams (i.e. visualized ECE) in Appendix E. 
We'd like to clarify that (see also L184-L195), calibration for confidence measures in *free-form* generation (as opposed to QA in a multiple-choice manner) is only somewhat related to our study. 
In general, a confidence measure could be considered good in *ranking* (higher score means an answer is more likely to be correct) and *level* (not overconfidence or underconfidence in general).
Our focus, like our baseline papers in free form NLG, is the ranking.
For example, the sequence likelihood itself is almost always very close to 0 for slightly longer sequences, due to the possibility of different phrasing of the same answer, so a post-hoc calibration method is required to make its level close to "P(correct)".
Therefore, if we measure the calibration quality, we are essentially measuring how good the post-hoc calibration method is (e.g. histogram binning, platt scaling, isotonic regression...), not the underlying confidence score.


## Minor comments and typos
$C_{LL}$ should be $C_{SL}$. Thank you for spotting this typo and we will correct it.
As for $\mathbf{s}_i$: could you kindly point us to where this is located? We use $s_i$ (e.g. Eq$(1)$) without bold font and might have missed this somewhere, but we cannot find the bold-font $\mathbf{s}_i$.
