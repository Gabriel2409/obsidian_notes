---
reviewed: 2023-08-01
---

#datascience

## Definition

In classification, we often need to go from the probability space to the log(odds) space.
log(odds) will be represented as

```text
7 samples: 4x and 3o
xxxx ooo
odds x = 4 / 3
log(odds) = log(4/3)
proba x = 4/ 7
```

## Going from probability to log(odds) and vice versa

We use the logit and logistic(= sigmoid) functions

$$l_o = logit(p) = log(\frac{p}{1 - p})$$

$$p= logistic(l_o) =  \frac{1}{1 +e^{-l_o}}$$
![[Logit_and_logistic_functions.png]]
