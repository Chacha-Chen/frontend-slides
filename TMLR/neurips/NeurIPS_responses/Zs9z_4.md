# Zs9z 

We appreciate your detailed feedback and comments. Before we move to addressing the review point-to-point, we would like to note that given the explosion of research in this area, it is quite impossible for us to try most (LLMs, NLI model, hyper parameter) combinations. However, we have tried to be thorough in what we could try. 


%The authors tried various different similarity scores and uncertainty/confidence estimates on top, some of which (esp EigV, Ecc, Deg) performing very similar. However, very little insightful experiments are provided in where the performance differences stem from. I would have preferred less similar methods and more detailed analysis. Especially because similarity seems more important. (Also see my question regarding other similarity metrics below)
### W1: Less similar methods and more anlaysis
First of all, we'd like to point out that EigV is only similar to Ecc and Deg in one experiment setting (Table 1).
Also, one of our findings is precisely that different ways to construct $C$ do not seem to matter as much as the similarity matrix itself. 
We appreciate the reviewer's suggestion on more analysis. 
Currently, we explain why NLI-based approach should be better in Section 4.1.
It is hard to pin-down the precise reason why entailment is better than contradiction, but we conjecture that this is related to the evaluation of the correctness of the answers - typically if an answer is too "far" from the gold standard answer, even if it is not contradicting, it is considered incorrect in QA. 
We will add this discussion to the experiment section.

%Several points regarding calibration and evaluation are unclear, see below, making it very hard to extract useful insights from the paper. Models are somewhat limited. I appreciate that using more models is costly (API use or hardware-wise), but the paper argues that we will have more black-box use in the future, so it would have been great to see a larger variety of models and move the smaller 13B models to the appendix.
### W2: More LLMs
When the paper was written, GPT-3.5 was the largest black-box LLM, and we included it in our experiments, and LLaMA was the SOTA open-sourced LLM. OPT was chosen as it was used in the SOTA baseline [23].
We appreciate the reviewer's understanding that the nature of this paper is not a comprehensive technical report of popular LLMs (which as the reviewer suggested is quite expensive, and even large industry labs might not be able to finish given the current rate at which new LLMs are proposed), but it is  a first step towards framing and tackling the problem at a reasonable scale. 
It is also unclear to us why being "black-box" implies the larger LLMs: Black-box in our context means that we get rid of the requirement to access logits or other information in prior works. 


%Some qualitative results would have been insightful to discuss, e.g., how do the constructed sets for NumSet look? How do the samples look? Is there enough variety among samples to start with?
### W3: Qualitative results
We are quite constrained by the space. 
Currently, some samples are provided in Figure 1.
We will add a few examples for the NumSet clustering in the additional page permitted for the camera-ready version if accepted. 
As for the variatbility of responses, it depends on the black-box LLM, and could be a limitation of many prior works - we will add this discussion (as a limitation) in the revision as well.

%There is no ablation in terms of hyper-parameters. Instead they are fixed, which seems counter intuitive if I were interested in uncertainty.
### W4: Fixed hyper-parameters
Instead of varying hyperparameters for the same model like [23], we chose to explore more models with their default parameters. 
We would appreciate if the reviewer could elaborate why this is counterintuitive as well.


%Metrics are also limited. ROC AUC is always a bit unclear it terms of usefulness and interpretation. So on top of the selective prediction results, metrics such as “is a good or the true answer included in the set” would have been interesting, alongside plotting confidence histograms or something like this across examples. Essentially, I am missing more analysis on top of the bare ROC AUC numbers the authors concentrate on.
%Table 1, 2 and 3 do not provide that much value to me. They are hard to read and barely discussed. Given they take up much space, more qualitative analysis or per-example analysis would have been more useful in my opinion.
### W5: Evaluation metrics
We agree that AUROC is a bit unclear, which is why we include AUARC as well, which has more concrete meaning in selective prediction (selective NLG is, however, our focus as stated in the abstract).
We include AUROC as it is the most popular evaluation metric for UQ.
Moreover, AUROC/AUARC are used very in different settings for $U$ and $C$, so Table 1 and 2 convey very different messages.  
We'd also like to point out that Table 4, Figure 1 and Figure 3 are not just AUROC numbers as well.
Could the reviewer kindly elaborate “is a good or the true answer included in the set”? We are not sure we follow exactly what the reviewer might have in mind. Is this pointing in the directon of prediction set / conformal prediction?
We will add confidence histogram in the revision.

> Table 1, 2 and 3 ... take up much space

We will move Table 2 or 3 to the Appendix, and add examples like Figure 1 for other metrics as well. 
In particular, we will provide the example embedding maps for Eccentricity.
We would like to note that much of the discussion is in setting up the exact meanings of these numbers (Section 5.2).
We will redirect some content currently in Section 5.2 to 5.4.




%I do not agree that epistemic uncertainty can “be reduced with additional information” (l111). In my understanding, this statement should be valid for aleatoric uncertainty. Epistemic uncertainty is reduced through better models, which the authors also highlight in their examples.
%In l125f, in my understanding, the confidence scores considered in this paper are purely empirical and do not bear any statistical meaning (in the sense of Bayesian or frequentist uncertainty estimations) – or this meaning is not made explicit theoretically.
### Minor comments 1: L111 on epistemic uncertainty
We understand that there are often slight differences in the usage of these terms. We follow [16] in page 2, which states
> epistemic uncertainty ... can in principle be reduced on the basis of
additional information

### Minor comments 2: L125 on confidence score
Your understanding is correct, and such literature typically cares more about the ranking (e.g. in [4,6,17,28]) of the score. 
For example, [17] uses a score that is distance based as well.
Distances could be linked to density if one is willing to assume parametric distributions, which is typically the case in Bayesian methods. 




%Conclusion: I am not sure if this paper is ready for NeurIPS. I feel the authors address an important problem and definitely have interesting starting points, but experiments lack insight and several key points (like samples, hyper-parameters, other similarity metrics) are unclear or underexplored. As a result, the experiments do not convince me to recommend acceptance.


%I am wondering how an alternative similarity metric based on embeddings and not specifically from a NLI model would perform. Depending on training of the NLI model, I am not sure if NLI is always the right notion to use here and simple embedding based similarity would provide the additional insight of knowing whether we actually need NLI or simple (semantic) similarity is sufficient.
### Question 1: NLI might not always be better
The reviewer is definitely correct in that the NLI model may not always be the best way to compute similarity. 
It is worth noting that even Jaccard similarities, which is only lexical and not even semantic, could help *when combined with the various $U$ and $C$ measures* and already can outperform the baselines. 
The use of embedding, however, is impractical in the black-box setting, and in fact also has been tried in [23]'s code and does not work.


%How would semantic sets perform when applied on top of the clustering (i.e., number of clusters after hard thresholding? There is a gap between NumSet and the graph-based uncertainty estimates and I am wondering whether this is partly due to NumSet being discrete and thus somewhat constrained.
### Question 2: Semantic Sets + Clustering

This is an interesting point. 
In our earlier attempts, we found that the discrete nature of semantic sec is indeed a big issue.
The continuous version improves upon the discrete version applied on top of spectral clustering.


%How exactly is accuracy evaluated for the different datasets – especially in light of there being multiple samples from the model. If NumSet, for example, is >1, this means there are multiple semantically different answers which will likely not all be true. But they all have been generated. So it is unclear how a label “correct” or “incorrect” is derived in order to have ROC and ARC plots.
### Question 3: How is accuracy evaluated
This is explained in Eq(10) and Eq(11), and it depends on which table you are referring to.
For confidence evaluation, each sampled answer has its own accuracy and confidence, so it's always paired (Eq. 10)
For uncertainty evaluation, we use the expected accuracy over all sampled answers (Eq. 11).
Note that NumSet is an uncertainty ($U$) measure, so it is the same for all sample answers.
We include uncertainty-only measures in Table 2 and 3 for comparison purposes, which might have cause confusion.
We will add clarification on this (similar to the caption of Fig. 2).


%The authors write that they calibrated the models on 1000 questions. This seems counterintuitive for uncertainty. Calibrating the temperature for test performance favors crisp decisions over uncertainty in order to maximize uncertainty. It will also limit the variety of samples. Can the authors elaborate on their thinking here?
### Question 4: Calibrating parameters
The temperature in L229 refers to that of the NLI model, not the LLM.
Due to our treatment of the LLM as a blackbox, the I/O consists only of natural language text, the temperature of the LLM is ``hidden``.
We will clarify this in the new version.


%Did the authors consider doing experiments with different prompting strategies? This is one of the big levers in a black-box setting.
### Question 5: Different prompting
No, we follow the same prompting as the LLaMA paper as well as [23] for fair comparisons. 
Different prompting is definitely an interesting topic but likely out of the scope of this paper. 