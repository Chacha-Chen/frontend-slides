# NzKW

## W1: Crowded Tables
Thank you for the suggestion - we will move the full tables to the Appendix and include only selected columns in the main text, and will point to the full tables. We will add some additional qualitative results as we have suggested to another reviewer.

## W2: Temperature
Thank you for the question.
There are two temperatures in question here (that of the LLM and of the NLI model) - in our paper, the temperature refers to that of the NLI model.
Thus, the outputs of the LLM are independent of the temperature tuning we mentioned in the experimental section.
However, although we treat the LLM as a black-box, as they also have an inherent temperature.
If the LLM temperature is set to very low, it is highly likely all methods we propose will fail and one would need to resort to a white-box method as the LLM is essentially deterministic.
We will add this to the limitation section of our paper.