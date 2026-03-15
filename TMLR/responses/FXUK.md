# FXUK

We thank the reviewer for the constructive review and acknowledging our contributions vis-a-vis experiments and methodology, as well as for providing useful feedback.
We would like to respond to some questions and concerns below. 

## Computation Cost

We agree with the reviewer that sampling adds overhead, and can be problematic in some use cases.
This is mostly due to the black-box model constraint, and many white-box methods indeed do not require as much sampling.
We would like to note that the sampling could be done in parallel (so latency is less of an issue than cost).
It would be interesting to explore methods that could reduce the number queries for probing black-box models.


## Sentence Embedding Model

This is an interesting idea, and it is definiteloy worth pursuing. 
While we haven't fully explored this idea due to limited time, we did notice that the baseline paper (Kuhn et al. 2023) has tried the sentence embeddings in their code release but didn't use it in the paper, and we assumed that sentence embedding-based method didn't work well.
Somewhat related to this, we have tried using word embeddings (Glove) to form sentence embeddings but it performs really poorly.
We think that while a good sentence embedder has the potential to perform well, it likely requires much more in-depth investigation. Consequently, we leave it for future work.




"requires" -> "require" : Thanks for catching this, we have corrected it. 