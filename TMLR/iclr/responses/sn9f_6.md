# sn9f

We thank the reviewer for the constructive review, and appreciating for our experiments and general motivation. We are also thankful for the useful feedback. 
We have made revisions to our manuscript as well as performed additional experiments following the reviewer's suggestions. 

## Uncertainty vs Confidence

Apologize for this confusion. 
To clarify, in short, $U$ is a property of the model's perceived posterior and depends only on $x$ (see L3-5 in Section 3.1, and Eq 1).
$C$ depends on both $x$ and the answer $\mathbf{s}$ (L2-L3 of Section 3.2 and Eq 2).
We have updated Section 3.2 to further clarify this.

> Confidence scores are defined based on the token probabilities, which are not available in black-box inference settings

Just want to clarify that Eq(1) and Eq(2) are only one possible uncertainty/confidence (that are however mainstream).
Exact definition is just anything taking the form of $U(x)$ and $C(x,\mathbf{s})$.
We hope the updated section 3.2 and the example helps clarify this.

## Non-QA NLP applications

(We assume "data-to-text" means tasks like captioning. Please let us know if this is a misunderstanding.)
Other NLP applications like data-to-text are definitely something we plan to explore next.
This paper mostly just follows existing UQ literature and uses QA as it is the easiest to perform evaluation on it. 
Evaluation in general is still quite hard.
In fact, we have seen very recent explorations on using model-predicted likelihoood as confidence scores on X-ray report generation task (but they also judge the quality of the report with ROUGE).

We believe that a lot of the ideas are transferrable and we would like note that CoQA is to some extent "data-to-text" except that the data is in textual form as well.
Our methods do not assume that the input has to be text, as we are only comparing the generations' similarities.
If the input is data, we think there are two possible solutions: 
We skip the data (image or audio) and directly use Jaccard or NLI model on the question and answers, OR;
We fine-tune a domain-specific NLI model by training a "bridging" module that "translates" the other modalities to embeddings that the NLI model understands.

## Why UQ for NLG is important
> In the introduction it's presented that LLMs are gaining attention -> it's crucial to quantify their uncertainty. After reading the paper, I still have to ask why? It seems crucial to improve their accuracy. Is the only rational for improving their uncertainty so that their accuracy can be improved, e.g. by blocking uncertain outputs? Or is there other reasons as well?

To begin with, improving the accuracy is important, but selective generation (i.e. blocking uncertain outputs) is not just about improving accuracy.
For example, an influential work [1] in selective classification (an area with a similar motivation, but with much more literature) aims at providing risk guarantees after the selection.

Secondly, if improving accuracy *on the un-rejected samples* is important, then finding a good UQ measure for such selective generation task is also important as a better UQ measure could lead to higher accuracy at the same rejection rate.
A bad UQ, such as the "Random" baseline in Table 1, leads to no improvement.

Finally, selective generation/classification is one of the most important applications of UQ, but in general UQ is about conveying this information about uncertainty to the decision maker (human or algorithm) [2].
In UQ literature, people tend to focus on AUROC to check whether the UQ measure is reliable, and we also use AUARC (a metric related to selective generation) as we think it is more directly related to real-world applications.
They are both machinery to probe the reliability of UQ measure, whose true importance is due to that decision-making processes intrinsically need to have a reliablity measure on the prediction. 

[1] Geifman, Yonatan, and Ran El-Yaniv. "Selective classification for deep neural networks." Advances in neural information processing systems 30 (2017).

[2] Seoni, Silvia, Vicnesh Jahmunah, Massimo Salvi, Prabal Datta Barua, Filippo Molinari, and U. Rajendra Acharya. "Application of uncertainty quantification to artificial intelligence in healthcare: A review of last decade (2013–2023)." Computers in Biology and Medicine (2023): 107441.

## UQ vs Calibration
> How is the issue of model calibration different?

(See end of Section 2 for a discussion, and a newly added Appendix C.5 for calibration experiments.)
The UQ we focus on tries to find a measure of uncertainty or confidence that is *discriminative* of what's uncertain and what's not. 
In other words, we care about whether the *ranking* of our UQ measure could effectively predict the reliability of the answers.
This is also why most literature uses AUROC as an evaluation metric.
Calibration cares about the discrepancy between the predicted probability and the frequency.
(Such "discriminative-ness" is often referred to as "sharpness" and is considered orthogonal to calibration [3].)
In the extreme case - if we know the model's accuracy is 0.6 on average, and we keep predicting a answer's confidence is 0.6 regardless of the answer, we get something that is prefectly calibrated, yet this constant confidence is not useful because it doesn't tell us which answer is more reliable than others (i.e. not "sharp").

[3] Gneiting, T., Balabdaoui, F., and Raftery, A. E. Probabilistic forecasts, calibration and sharpness. Journal of the Royal Statistical Society: Series B (Statistical Methodology), 69 (2):243–268, 2007.