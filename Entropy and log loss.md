---
reviewed: 2023-07-30
---

#datascience #todo
https://youtu.be/ErfnhcEV1O8

## Entropy

- Entropy measures the unpredictability of a probability distribution. The more random, the more the entropy
- Uniform distribution = very large entropy
- Entropy can be seen as the reduction of uncertainty when you get the information
- ex: 8 equiprobable events. If you get the confirmation of an event, you get 3 bits of useful information. Associated entropy is inverse of proba

## Cross entropy

- mesures diff between real distribution and predicted distribution
- Can be seen as the average message length in bits

## Kullback- Leibler divergence

- diff between cross entropy and entropy

## Log loss

- Also called cross-entropy loss
- Same formula as cross entropy but uses natural logarithm instead of base2 logarithm
- often used in ML classification problems as a loss function
