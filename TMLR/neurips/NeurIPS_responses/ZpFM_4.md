
# ZpFM

%The spread in semantic similarity (which the authors use as their metric of model uncertainty) could be due to aleatoric or epistemic uncertainty. For the setting discussed by the authors (whether or not to abstain from generation), epistemic uncertainty seems much more relevant. I would have appreciated a deeper discussion of this
### W1: Epistemic vs Aleatoric Uncertainty
In general, both uncertainty (i.e. the total uncertainty) are important. 
In the case of QA where we use "how well uncertainty/confidence predicts accuracy" as the evaluation metric, epistemic uncertainty does seem more relevant, because these questions are relatively well-defined.
We will add such discussion to L109-L122.


%Intuitiuvely, semantic similarity does not seem like a comprehensive metric of uncertainty to me. For example, I could imagine that “Olympia” and “Corinth” are very semantically similar concepts under a model even though they are different places.
### W2: Olympia vs Corinth 
We agree that it also has its limitations. We'd like to clarify that we do not argue semantic similarity is already the comprehensive solution to this problem - instead, we improve the ideas in the recent ICLR spotlight [23] to black-box settings and improve the performance. Nevertheless to a first approximation, we think that it captures at least some sense of how uncertainty is perceived in natural language generation. 
We currently tune the temperature for NLI in order to capture this "how different is different" question (L228). 
%Further, we could in principle have a task-specific NLI model which could be better at distinguishing finer or coarser shades of semantic meaning (e.g. in your example, you seem to be suggesting that if the larger context is 'Greco-Roman' then these concepts are similar, and if the context is more fine-grained, then they are not similar). One can also imagine that one could improve such models' classificatoin based on additional context. This is an interesting direction of research that currently is perhaps outside the scope of our paper.

%The experimental section was very hard to follow. It was never explicitly stated how the quality of the uncertainty/confidence metrics was quantified. Something as simple as stating: we take a metric’s quality to be our ability to use them to predict the accuracy of a model’s response. I also didnt understand how the “Oracle” (maximum) AUARC was computed. What does it mean to use the “target (accuracy) as the rejection threshold?”
### W3 and Q1: How the quality of U/C are measured
We apologize for the confusion. We will try to clarify some of the points raised. 

> It was never explicitly stated how the quality of the uncertainty/confidence metrics was quantified

Currently the exact formulas are given in Eq.(10) and Eq.(11). 
Eq.(10) corresponds to Table 2 and 3, and Eq. (11) to Table 1.
It is indeed like you said - using confidence to predict accuracy (L260 and L263). 
This is the same as the AUROC evaluation in [23]. Following your comment, we will try to clarify this further so that it is very explicit. 

> how the “Oracle” (maximum) AUARC was computed

Rejection threshold refers to the threshold to filter samples before computing the accuracy - for example, one might say we "reject" all responses whose uncertainty is higher than $T$.
We vary this $T$ to draw ARC and thus compute AUARC.
In other words, if we reject the high-uncertainty samples by treating the (negative) expected accuracy as the thresholds, we achiveve the highest-possible AUARC.
Mathematically, in Eq.(11), imagine we have $N$ samples' expected accuracy (not binary, since they are the average of 20 samples' accuracy).
WLOG assume $\mathbb{E}[acc_1] \leq \mathbb{E}[acc_2] \leq \ldots \mathbb{E}[acc_N]$.
The $U$ measure gives the highest possible AUARC when $-U(x_1) \leq \ldots -U(x_N)$.
In other words, if we cheat and use the expected accuracies themselves to predict themselves, we achieve the highest AUARC.
Due to the space limit we could fully explain AUARC (which took a full paper to study [33]), but we will clarify what this rejection threshold means. In the paper is accepted, the additional page will give us leeway to explain these in more detail, which we will do. We will also take your comment as a lead to improve the description in our experimental section. 





%While the authors themselves state that uncertainty and confidence are not antonyms, they use them as such regularly throughout the paper and never explicitly “differentiate the two closely-related notions”, as advertised in the abstract

### W4: Explicit differentiation of uncertainty and confidence

We have done this in Section 3.2.
But to clarify, uncertainty depends only on $x$, but confidence depends also on the particular answer $\mathbf{s}$.

### Q2: Additional Literature
Thank you for pointing out this additional reference. We had somehow missed it. We will include it in our revision.