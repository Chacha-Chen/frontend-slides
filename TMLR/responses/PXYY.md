# PXYY

We thank the reviewer for the positive review, and appreciating for our experiments and writing. 
We are also thankful for the very detailed feedback. 
We have made revisions to our manuscript as well as performed additional experiments following the reviewer's suggestions

## Min Instead of Mean
Thank you for the suggestion. 
We experimented with the suggested version for "entail", and observed that the "min" outperforms the "mean" in two out of 12 columns in Table 1 and 2, and are otherwise worse, but often statistically insignificant. 
We think the "min" version is reasonable, and our original motiviation was simply to reduce the noise and make the similarity matrix symmetric.
The empirical difference could be due to some implicit human-level overfitting, but is likely due to the nature of the QA task - the ideal similarity could be something in between "entail" and "no contradiction".
That is, "entail" might be too strict, and "no contradiction" might be too loose.
Thus, "mean" might implicitly be affecting a small trade-off by making the "entail" similarity a bit less strict.



## Discussion Regarding Open-ended Questions

Thank you for raising this concern and we totally agree with the reviewer that these are also important questions to be explored. 
We realized that the tone of our original writing could be misleading, and have updated this section in the revision. 
Our point was that the evaluation of UQ in the open-ended question setting is itself a challenging research, so our work only intended to use the more well-defined evaluation framework for QA for our proposals.


## Typos

Thank you for the careful reading and identification of these typos! We have corrected them in this revision. 
Regarding typo 4 - we used LLaMA and triviaqa for all plots. 
If we look at confidence measures created by 20 samples, we have 20 AUARCs (one of each time we sample).
It just happens that for trivia+LLaMA, the first 2-3 are a bit higher than the rest, so the tread looks downwards as we take the average of the first 3, 5, 10, and 20 generations.
In some sense, this is like a random walk (along $m$) with no drift that exhibits a spurious trend (for Coqa+LLaMA this looks upward-trending, for example), and confidence measures tend to achieve higher AUARC so the positive drift erases such random trends.
We've clarified this in the revision.

