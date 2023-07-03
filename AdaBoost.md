#datascience

## Definition

- Contrary to [[Random Forest]], in a forest of tree with AdaBoost, trees are usually just a node and two leaves = stump (weak learner)
- In a random forest, each tree has an equal vote on the final classification. In contrast, in Adaboost, some stumps get more say in the final classification than others
- In a random forest, each tree is made independently of the others. In AdaBoost, order is important; the error that a stumps makes influences how the next stump is made

## Building the forest

- To build trees, each sample is given a weight: The weight indicates how important it is that the sample is correctly classified
- For a given tree,
  - `Total error = sum of the weights associated to incorrectly classified samples`
  - `Amount of say = 1/2 * log([1 - Total_error]/ Total_error)`
- To create the first stump, we give equal weight to each sample. So the process is the creation of a simple decision tree.
- To build the next tree and all subsequent trees, we increase the weight of the incorrectly classified samples and decrease the others.
  - For incorrectly classified: `new_weight=old_weight *e ^ amount_of_say`
  - For correctly classified: `new_weight=old_weight *e ^ - amount_of_say`
  - Then we normalize the sample weights so that they add up to 1
- In theory we could use the sample weights to calculate weighted gini indexes to determine which variable should split the next stump.
- Alternatively
  - we start by making a new empty dataset (same size as the original)
  - Pick a number between 0 and 1 and see where it falls if you use the sample weights like a distribution
  - Add the corresponding sample
  - Repeat until new collection is the same size as previous (note that same sample can appear several times)
  - Finally we give all of the new samples equal sample weights to build the next stump
  - Note that samples that were not selected retain their weights for the next iteration

## Classification process

To make a prediction, sum the amount of says for each stump predicting a given category. The final prediction corresponds to the category with the highest sum
