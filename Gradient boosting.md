#datascience

## Definition

- Gradient boosting uses an ensemble of [[Decision tree]] for its prediction
- Like [[AdaBoost]], gradient boost build fixed sized trees based on the previous tree’s errors, but unlike Adaboost, each tree can be larger than a stump (in practice, nb of leaves is set between 8 and 32)
- Contrary to Adaboost, each tree has the same weight (the learning rate)

## Building for regression

- Create the root of the first tree = average of the target values
- For each sample, save the diff with average (pseudo residual)
- Now we will build another tree to predict the RESIDUALS instead of the target
- Because we restrict the nb of leaves, several samples will appear on the same leaf: we use the average of their residuals
- We can combine the first tree with the new tree to predict the target values. To avoid overfitting, the new tree is scaled with a learning rate

- Now we recalculate the pseudo residuals = diff with our latest prediction and use the same method to build another tree. This tree is also scaled by the learning rate

## Building for classification

- Root of the first tree is log(odds). Note: log(odds) can be thought as the logistic regression equivalent of the average
  - For ex, if you have 4 samples whose target is true and 2 whose target value is False, `log(odds) = log(4/2) = 0.69`
- We can convert the log(odds) to a probability: `prob(True) = (e ^ log(odds))/(1 + e^ log(odds)) = 0.66`

- Below we represent the classification problem as if it was a regression problem
  - Middle dotted line corresponds to previous step probability
  - Residual is difference between sample position and line
  - x axis legend correspond to target value

```text
  |
1-|        xxxx
  |_____________
  |
0-|  xx
  |--|-------|-
 	False    True
```

- To calculate the residual in the probability space, we compute the diff between the target value (0 or 1) and the probability calculated in at the previous step (here 0.66)
- Now we build another tree to predict the residuals
  - Contrary to regression we can't just add the residuals in the leafs as the prediction is in the log(odds) space while the residuals is in the probability space
  - So we must first transform this probability residual into a log odds residual
  - Final value is `Sum over samples in leaf of Proba Residuals / Sum over samples in leaf of (prev proba)(1- prev proba)`
  - The new tree is scaled with a learning rate to get the new log(odds) prediction for all samples. Then we derive the associated probability and we can continue
