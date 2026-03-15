## GpdJ

We appreciate the constructive feedback and are glad that the reviewer found our motivation and method intuitive and interesting.
However, we would also like to take this opportunity to respond to some concerns regarding the clarity/presentation of our work. 


## Semantic Similarity Measures

It is definitely true that it is OK to use an independent model for similarity measurement.
There are a few reasons why we include Jaccard and NLI.
Since it is unlikely that we could exhaust a large suite of models to perform similarity measurement, we chose Jaccard and NLI as simple representatives of methods that rely solely on lexical similarity vs methods that explore the semantic meanings. 
Another reason for using NLI was that we wanted to ensure an apple-to-apple comparison with the baseline paper (Kuhn et al, 2023) as well.

## Similary Performance of Deg,Ecc, and EigV

Unfortunately, it is much harder to provide explanations when some methods perform similarly instead of differently, but we'd like to provide a few comments and conjectures below. 
First of all, we would like to point out that the performance is similar in the uncertainty setting, but Deg and Ecc can identify more confident responses while EigV cannot.
However, it is indeed true that in many cases, these three constructions give similar performance. 
We suspect that there is a limit given the similarity matrix, and after the hyperparameter tuning on the calibration set, Deg/Ecc/EigV almost extract all useful similarity information. 
While the three constructions perform similarly in AUROC/AUARC, we think each construction has advantages unrelated to performance: EigV is quite interpretable, Deg is the simplest, and Ecc could provide embeddings for each response.

## Confidence/Over-confidence

We would be grateful if the reviewer could elaborate why overconfidence makes Eq(2) misleading? From the provided comment this is not quite clear to us. Nevertheless, a few comments are in order below:  
We include a discussion related to overconfidence in the last paragraph of Section 2. 
It is quite likely and often the case that a confidence measure (whatever it is) is mis-calibrated and over/under-confident. 
Eq (2) is only an example of confidence score that was widely used in literature, could likely be miscalibrated, and is not *the* confidence score to use.
The point of Section 3.2 (Uncertainty vs Confidence) is to distinguish two concepts - one related to the predictive *distribution*, and the other a specific response/prediction. 
We have also provided experiment results on calibrated confidence measures in the Appendix (Table 16), which suggests that if miscalibration is a concern, off-the-shelf calibration methods could easily be combined with any confidence measures, but a good confidence measure should provide good *ranking* of the answers.