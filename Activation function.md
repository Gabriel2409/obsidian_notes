#datascience

An activation function is a mathematical operation applied to the output of each neuron (or unit) in a neural network layer. It introduces non-linearity into the network, allowing it to learn complex patterns and relationships in the data.

## Examples

![[Activation function - main]]

### Sigmoid

The sigmoid function (or logistic function) squashes input values into the range between 0 and 1.
The sigmoid function is commonly used to map the output of a neural network to a probability value, especially in binary classification
$\sigma(x) = \frac{1}{1 + e^{-x}}$

Note: opposite function is logit function: $\text{logit}(p) = \log \left( \frac{p}{1 - p} \right)$

### Softmax

Softmax can be seen as a generalisation of sigmoid for multi-class classification tasks.
While the sigmoid can be seen as outputting the probability of a given class in binary classification, the softmax produces multiple values corresponding to the probability of each class, while ensuring that the probabilities sum to one.

If we have a vector of $K$ real-valued scores (logits) denoted as $z = [z_1, z_2, ..., z_K]$ where $z_i$ is the score for class i, $\text{softmax}(z_i) = \frac{e^{z_i}}{\sum_{j=1}^{K} e^{z_j}}$

### ReLU

ReLU (Rectified Linear Unit) induces sparsity in neural networks by zeroing out negative values in the input, , effectively reducing the number of active neurons in the network. This can lead to more efficient learning and better generalization, as it encourages the network to focus on the most relevant features.

Moreover, ReLU is easy to compute and avoid mitigate the vanishing gradient problem by allowing gradient to flow through positive values.

$\text{ReLU}(x) = \max(0, x)$

ReLU has a lot of variants. For example, Leaky ReLU allows a small, non-zero gradient for negative inputs, thus addressing the "dying ReLU" problem observed in some deep neural networks.

$$
\text{LeakyReLU}(x) = \begin{cases}
x & \text{if } x \geq 0 \\
\alpha x & \text{otherwise}
\end{cases}
$$

