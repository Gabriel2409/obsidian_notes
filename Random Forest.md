#datascience

## Definition

A random forest is an ensemble learning method that combines multiple [[Decision tree|decision trees]] to make more accurate predictions.

To build a tree:

- Step 1 : create a **bootstrapped** dataset: We randomly select samples from our original dataset so that our bootstrapped dataset has the same size. Note that we can select the same sample several times
- Step 2 : Create a decision tree using the bootstrapped dataset but only use random subset of variables at each step

The forest is built by creating several of these trees. To classify a sample, we run it through all of the trees and get the value that has the most votes.

Notes:

- Using a bootstrapped dataset and then the aggregate is called **bagging**.
- Using a bootstrapped sample and considering only a subset of the variables at each step results in a wide variety of trees; The variety is what makes random forests more effective than individual decision trees.
- Because we used bootstrapped datasets, some of the examples were not on the tree. We can measure how accurate the random forest is by the proportion of **out-of-bag** samples that were correctly classified
- The proportion of out-of-bag samples that were incorrectly classified is the **out-of-bag** error. Note that out of bag samples are only run through trees that don’t use them to be built.
