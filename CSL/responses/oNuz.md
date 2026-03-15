# oNuz

We thank the reviewer for the positive appraisal of our approach and distilling its benefits. We are also thankful for the constructive feedback, which we engage with point by point below. 



## Weaknesses
We indeed limit our experiment to QA tasks, primarily due to clarity afforded by simplicity in its evaluation.
As a comparison, while the confidence might indicate the quality of the translation/summary, the "quality" itself it quite difficult to measure, making the evaluation of the confidence measures difficult as well.
We think this might be the reason why many recent works focus on QA datasets for uncertainty quantification in general.

We also agree with the reviewer regarding the limitations due to the prompt.
However, CSL-Next does not use any prompt but also performs better than SL/SL(norm). We find this encouraging, somewhat surprising, and think that it partly addresses this concern.
That said, we do believe a tailored prompt could potentially give better results in general.

## SE
We appreciate the question. 
We would like to clarify that SE stands for "Semantic Entropy" and refers to clustering the semantics of mulitple generations before computing entropy (using their sequence likelihoods) (Kuhn et al 2023). 
Kuhn et al 2023 tried using both normalized and unnormalized sequence likelihoods, and proposed to use different versions for different datasets, which is hard to decide a priori.
Table 4 essentially uses CSL in their pipeline (to replace the vanilla sequence likelihoods in $p(\mathbf{S}|x)$ in their Eq(2)). 
Thus, Table 4 is about improving uncertainty (from the left 2 columns to the right-most column) by using CSL. Please let us know if this answers the question as well.


## Non-autoregressive Models
We thank the reviewer for this interesting question.
We limit our study to autoregressive model because the sequence likelihood has a nice interpretation (log of product of conditional probability) and is thus heavily used as a confidence score in prior literature.
Another consideration is their popularity in recent years.
We think for encoder-decoder only models, QA tasks are often framed as classification tasks, which makes our proposed method less applicable.
However, reweighting the token logits could probably improve the confidence quality in tasks like machine translation. 
A potential challenge there is that without a prompt it might be harder to elicit a good attention weight---we might need to resort to some combination of part of speech classification and NER techniques. We believe this could be a fruitful avenue for future work.


