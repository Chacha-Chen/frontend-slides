# YpC9


We thank the reviewer for the constructive review and acknowledging our contributions vis-a-vis experiments and presentation, as well as for providing useful feedback.
In pursuance to the suggestions made by the reviewer, we have made revisions to our manuscript, and performed some additional experiments.

## ECE or ACE (calibration measures)

Thank you for the suggestion.
We tried to distinguish the focus of our paper at the end of Section 2.
Just like in our baseline papers, we focus on the uncertainty estimation, and we care more about the *ranking* of such measures - that is, whether a high confidence means highly reliable prediction or accuracy.
All uncertainty or confidence estimates could be conformalized or calibrated to bear concrete meaning in the frequentist sense, by using a calibration dataset.
There is a large collection of calibration literature that we can leverage here. 
In repsonse to your suggestion, we have also run additional experiments using histogram binning to calibrate all methods, and report the ACE in the Appendidx C.3. 

## Additional Reference
Thank you for the suggestion and we have included this reference to our revision for completeness.
After reading the paper, we found that this method is more suitable for questions with exact answers (e.g. multiple choice or arithmetics) as the consistency-based component requires comparing exact matches of predictions, which is quite hard to achieve for examples like Figure 6 in the Appendix.
The "verbalized confidence" is similar to the P(true) baseline we tried.

## Use of model embeddings

> Would this method (using model embeddings) give better confidence or uncertainty estimates in those settings as well?

Thank you for the question.
While we did not perform the experiments ourselves (and is not a black-box method), we noticed that our baseline paper's (Kuhn et al. 2023) official implementation tried this idea.
As it was not used nor reported in their paper, it is probably not performing well.
