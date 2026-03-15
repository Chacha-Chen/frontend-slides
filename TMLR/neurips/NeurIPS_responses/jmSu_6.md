# jmSu

%From the introduction, it seemed that authors would focus on a downstream application of the uncertainty quantification methods as test bed for their quality (i.e., selective generation). However, this is not a well developed idea in the paper (only Table 4 includes results for that setting, and very broadly). This should have been one of the main foci of the method's evaluation since it is arguably the best evaluation setting at hand, in my opinion.
### W1: Evaluation in selective generation
Thanks for your comment. However, to elaborate, there are two types of selective generation.
We mentioned the literature we base on (selective classification/classification with rejection) in L73-L84 and L125-L127.
In selective classification, the selection is done by rejecting inputs - that is, we find some score basing on which we refuse to classify some images, for example.
In our opinion, a direct equivalent in NLG would be rejecting the answer from the LLM for certain inputs.
This is also why the arguably few papers on the topic such as [23] uses AUROC as the evaluation metric.
Therefore, Table 1-3 are all selective NLG as well.
In a different but somewhat related task, https://arxiv.org/pdf/2209.15558.pdf also adopts this interpretation of "selective generation" and their "QA curve" is similar to ARC (L248 in our paper).
However, it is possible that we misunderstood and would greatly appreciate if the reviewer could elaborate what he/she means by selective generation in this comment. We will reword accordingly to ensure that the narrative doesn't seem misaligned. 

%I believe that space could have been better distributed in the paper. Currently, the section where results are discussed (sec. 5.4) takes about 1/2 a page only. That means there could be space for a better contextualisation and discussion of the results presented in different tables.
### W2: Better distribution of space
Thank you for the suggestion. 
We were heavily constrained by the space.
As we replied to reviewer Zs9z, we plan to dedicate much of the additional page for discussion around the results. 

%Results are reported on a number of QA datasets with different proposed methods, but other NLP tasks could have also been investigated. Also, Tables are very dense and with lots of results, but the significance of those many results could be better discussed in the results section.
### W3: Other NLP tasks
Thank you very much for the suggestion.
We mostly follow and extend [23] to more models and datasets.
It is definitely interesting to see how our metrics extend to other NLP tasks, and we are actually actively investigating it right now.
However, due to the space, we decide to focus on the QA task.
We will modify our abstract to emphasize this focus as well.

> Also, Tables are very dense and with lots of results, but the significance of those many results could be better discussed in the results section.
We respond to this in Q3 and Q4.

%The procedure in lines 182-185 assumes that the prior probability for entailment and contradiction are similar, but I don't think that is necessarily the case. In practice, there should be different amounts of entailment/contradictions in the NLI training data (assuming the model is well calibrated). Would this procedure still make sense if that requirement is not met?
### Q1: Merging Sets in NumSet
The reviewer mentioned a very good point and we agree.
However, NumSet is not our method, but taken from [23] as a baseline.
We think it is limited and generalizes with L186-L206.


%l.227-230: how large is your "calibration set" (and, by consequence, test set)?
### Q2: Size of calibration set
The sizes are mentioned in L225-L228.
The calibration set is always 1000, making the test set 6983/2610/8960 for coqa/nq/trivia.

%In your tables, you are making a single entry bold-faced based solely on the means but not on the standard deviations. Shouldn't you bold-face all best results that overlap with mean-SD of your highest scoring method? Otherwise, you are not using the standard deviations for anything, and reporting them doesn't really add much. Do you agree?
### Q3: The use of standard deviation
The standard deviation is indeed not currently used in the formatting, but just provided as an additional information for readers.
In the past (other papers) we bold-face numbers basing on p-values, but in this particular case due to the overlap of samples in each experiment the p-value is hard to compute.
We will add underscores to methods that are not different from the best method according to the overlap of mean-SD as the reviewer suggested.

%From the results on tables, it is clear that the methods that use NLI-contradiction or Jaccard similarities consistently underperform the NLI-entailment method. Why not move these other results to an appendix, and focus on the part that is the most promising in the main paper? It would be nice to add details about using your models for selective generation, and how much you can improve performance there.
### Q4: Moving Jaccard and NLI-contradiction to Appendix
This is a great suggestion. We will keep one table with all similarities and remove those columns for other tables in the revision and move things to the Appendix. 

> It would be nice to add details about using your models for selective generation,

We think you are referring to Table 4. If you mean something else, please let us know and we will do the needful. 