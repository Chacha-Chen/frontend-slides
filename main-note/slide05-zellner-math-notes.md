# Slide 5: Zellner's Delta — Math Intuition (Speaker Notes)

## The Formula

$$\Delta(q) = \underbrace{D_{KL}(q(\theta|X_{1:n+1}) \| \pi(\theta|X_{1:n}))}_{\text{Term 1: Info out}} - \underbrace{\mathbb{E}_q\left[\log \frac{\ell(X_{n+1}|\theta)}{m(X_{n+1})}\right]}_{\text{Term 2: Info in}}$$

## Term 1 — "How much did your beliefs actually shift?"

- KL divergence from prior to posterior
- Measures the size of the belief update after seeing new evidence

## Term 2 — "How much *should* your beliefs have shifted?"

- Posterior-weighted average of how informative the new evidence was
- Expands to: $\int \log \frac{\ell(X_{n+1}|\theta)}{m(X_{n+1})} \cdot q(\theta|X_{1:n+1}) \, d\theta$
- In plain English: for each hypothesis θ, compute how informative the evidence was at that θ, then take a weighted average where the weights are how much the model believes each θ

### Breaking down the ratio inside the log:
- $\ell(X_{n+1}|\theta)$ = how likely the new evidence is under a specific hypothesis
- $m(X_{n+1})$ = how likely the new evidence is on average (across all hypotheses, weighted by current beliefs). This is $p(X_{n+1}|X_{1:n}) = \int \ell(X_{n+1}|\theta) \pi(\theta|X_{1:n}) d\theta$, so it DOES depend on previous evidence
- The ratio = how much the new evidence favors this specific hypothesis over the average

### What does $\mathbb{E}_q$ (expectation under posterior) do?
- It's a weighted sum where the weights (posterior) sum to 1 — so it's equivalently a weighted average
- Weights = posterior beliefs $q(\theta|X_{1:n+1})$
- Hypotheses the model considers more likely contribute more to the information measure
- Discrete example: if model thinks 3 diagnoses have probabilities [0.7, 0.2, 0.1], then: 0.7 × (info at diagnosis 1) + 0.2 × (info at diagnosis 2) + 0.1 × (info at diagnosis 3)

## The Punchline

**Delta = (how much you changed) − (how much you should have changed)**

- Δ = 0 → update matched evidence strength exactly → **this IS Bayes' rule**
- Δ > 0 → over-updated (shifted beliefs more than the evidence justified)
- |Δ| > 0 → deviation from optimal; larger = worse reasoning

**Key insight:** Delta is a self-consistency check. It doesn't compare against external ground truth. It asks "did the model contradict itself?" — did it update by an amount inconsistent with its own assessment of how strong the evidence was?

## Doctor Analogy

If a doctor gets a highly diagnostic test result, they should update a lot. If they get an uninformative result, they shouldn't. Term 2 quantifies this "should" — and Delta measures whether the model respected it.

## One-Sentence Summary

Did the model shift its beliefs (KL divergence) by the same amount as the posterior-weighted average informativeness of the new evidence?
