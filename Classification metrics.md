---
reviewed: 2023-08-02
---

#datascience

Binary confusion matrix. Note that I put the True class first. If you want the same with the class False first, just invert the n and the p

![[Confusion_matrix_bin.excalidraw]]

$$
\begin{align}
precision = PPV = \frac{t_p}{t_p+f_p} \\
NPV = \frac{t_n}{t_n+f_n} \\
recall = sensitivity = TPR = \frac{t_p}{t_p+f_n} \\
specificity = TNR = \frac{t_n}{t_n+f_p} \\
FNR = 1 - TPR = \frac{f_n}{t_p+f_n} \\
FPR = 1 - TNR = \frac{f_p}{t_n+f_p} \\
F1 = 2 \frac{precision * recall}{precision + recall}
\end{align}
$$

- **Precision = How many retrieved items are relevant** =proportion of predicted items that are actually correct. For Binary classification: $precision = \frac{t_p}{t_p+f_p}$
- **Recall = How many relevant items are retrieved** = proportion of predicted items among all correct items. For Binary classification: $recall = \frac{t_p}{t_p+f_n}$

- precision is also called PPV (positive predictive value). Note that if we measured the prediction of the "False" class, we get the NPV: $NPV = \frac{t_n}{t_n+f_n}$
- recall is also called sensitivity. Note that if we measured the recall of the "False" class, we get the specificity: $specificity = \frac{t_n}{t_n+f_p}$
- recall is also called True positive Rate (TPR) and specificity is also called True Negative Rate (TNR)

- ROC (Receiver Operating Characteristic) curve = TPR vs FPR as threshold changes. It shows the sensitivity / specificity (1- FPR) tradeoff
- Random classifier: ROC is a line., AUC (area under curve) is 0.5
- Perfect classifier: AUC = 1

- ROC-PR: same but with précision recall curves

- Aggregation
  - Macro: give each class the same weight in the final score
  - Weighted: weight each class by its support
