We are thankful to ARR for delivering a constructive and positive assessment of our paper, well-summarized in the meta-review. We are glad that the reviewers and the meta-reviewer found our submission to be simple and effective, well-evaluated (for both the range of LLMs and data and ablations) and potentially useful for calibrating LLMs.

We would like to briefly discuss the suggested revisions below:
 

> The experiments are focused solely on Question-Answering tasks, leaving its performance on other datasets unclear.

We focus on question-answering tasks since it affords fair and objective evaluation easily. 
For other tasks which involve more free-form answers, the main bottlebeck is the evaluation itself (which could involve significant human invervention). 
How to do this efficiently is an open research question by itself, hence we decided to evaluate our confidence measures on tasks where evaluation is broadly agreed upon. 
We will add this discussion to our revised manuscript.

> The authors address ambiguity through prompting, but this method is generally effective only for task-specific datasets. Its effectiveness for other tasks is uncertain. However, the strong performance of CSL-Next, which does not use any prompting, is promising and partially alleviates this concern.

As the meta-reviewer notes, CSL-Next partially alleviates this concern. 
While any definitive conclusion could only be based on systematic experiments, for other tasks, we believe the attention induced by a custom prompt, even with little to no optimization, could outperform the naive attention from CSL-Next. 
We have discussed this in the Limitations section, but will make it more prominent.

