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

`Predicted_value = Root_tree + lr * Tree1_pred + lr * Tree2_pred...`

## Building for classification

- Steps are mostly the same but the difficulty is that you move from the probability space to the log(odds) space (see [[Odds, log odds and probability]]). 
	- Residuals are calculated in the probability space
	- Predictions are made in the log(odds) space

- Root of the first tree is log(odds) which will act as our average in regression
- Then, we convert it to a probability: $p = \frac{e^{log(odds)}}{1 +e^{log(odds)} }$ and use this probability to calculate the residuals (note that, for the first iteration, p can also be calculated directly)
- When building our trees, we try to predict the residuals in the **probablity space**, so to get the final value of a leaf in the **log(odds)** space:  `Sum over samples in leaf of Proba Residuals / Sum over samples in leaf of (prev proba)(1- prev proba)`

$$
\gamma_{jm} = \frac{\sum_\limits{x_i\in{R_{jm}}}r_{im}}{\sum_\limits{x_i\in{R_{jm}}}p_{m-1}(x_i)(1-p_{m-1}(x_i))}
$$

- The new tree is scaled with the learning rate and we can then compute the current log(odds) for a given sample, that we then convert to probability and use this value to calculate the next residuals


## Mathematical approach

### Overview

Step 1: initialize model with a constant value $F_0(\textbf{x}) = \underset{\gamma}{\mathrm{argmin}}\sum_\limits{i=1}^{n} L(y_i, \gamma)$
Step 2: For m = 1 to M:
- 2a: Compute so-called pseudo residuals (the minus sign allows to use positive learning rate)
$$
r_{im} = -\left[\frac{\partial L(y_i, F(x_i))}{\partial F(x_i)}\right]_{F(x)=F_{m-1}(x)} \quad \mbox{for } i=1,\ldots,n.
$$
- 2b: Train regression tree with features x against r and create terminal node regions $R_{jm}$ for $j=1,...,J_m$ 
- 2c: Compute $\gamma_{jm} = \underset{\gamma}{\mathrm{argmin}}\sum_\limits{x_i\in{R_{jm}}}L(y_i, F_{m-1}(x_i) + \gamma)$ for $j=1..J_m$
- 2d: Update the model: 
	- For a given $x_i$, find the corresponding terminal region $R_{jm}$ and update the prediction: $F_{m}({x_i})=F_{{m-1}}({x_i})+\nu\gamma _{jm}$ 
	- Which can be rewritten in vector form: $F_{m}(\textbf{x})=F_{{m-1}}(\textbf{x})+\nu\sum_\limits{j=1}^{J_m}\gamma _{jm}1(\textbf{x}\in{R_{jm}})$ 

### Regression details

-  $L(observed, predicted) = (observed - predicted)^2$
- Step 1: to get the constant value $\gamma$ associated with the prediction:
$$
\begin{align}

\sum_\limits{i=1}^{n}\left[\frac{\partial L(y_i, \gamma)}{\partial \gamma}\right] = 0 \\

\implies \sum_\limits{i=1}^{n}(y_i - \gamma) = 0 \\

\implies \gamma = \frac{1}{n}\sum_\limits{i=1}^{n}y_i
\end{align}

$$
That's how we get the average for the first estimation

- Step 2a: For each of the sample, the residual becomes the difference between observed and predicted: $r_{im} = y_i - F_{m-1}(x_i)$
- Step 2c: to get $\gamma_{jm}$:
$$
\begin{align}

\sum_\limits{x_i\in{R_{jm}}}\left[\frac{\partial L(y_i, \gamma_{jm})}{\partial \gamma_{jm}}\right] = 0 \\

\implies \sum_\limits{x_i\in{R_{jm}}}(y_i - F_{m-1}(x_i) - \gamma_{jm}) = 0 \\

\implies \gamma_{jm} = \frac{1}{p_m}\sum_\limits{x_i\in{R_{jm}}}r_{im}
\end{align}
$$
with $p_m$ the nb of samples in region $R_jm$

That's how we show that the final value in each region is the average of their residuals

- Step 2d: For each sample, we compute the new pseudo residual


### Classification details

- $L(observed, predicted) = -[observed*log(predicted) + (1-observed)*log(1-predicted)]$
- For the first step, we will derive it with regards to the log(odds) which gets us $\gamma = p$
- The next steps is quite math heavy but we get $\gamma{jm}$ as shown above
