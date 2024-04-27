#datascience

![[Transformers - architecture]]

## General concepts

The [Transformer architecture](https://arxiv.org/abs/1706.03762) was introduced in June 2017.
Transformers are trained as language models (self supervised training on a large amount of data).

To adapt a transformer to a specific usecase, it is common to perform fine-tuning, which means reusing a pretrained model and performing an additional task on a specific dataset.

A transformer is composed of an encoder (left in the picture) and a decoder (right in the picture). These components can be used together or separately.

The encoder receives an input and builds a representation of it (its features). It is best suited for tasks requiring an understanding of the full sentence, such as sentence classification, named entity recognition (and more generally word classification), and extractive question answering. For example, BERT is an encoder

The decoder uses the encoder’s representation (features) along with other inputs to generate a target sequence. It is best suited for tasks involving text generation. For example GPT-3 is a decoder.

Encoder-decoder models (full transformer architecture) are best suited for tasks revolving around generating new sentences depending on a given input, such as summarization, translation, or generative question answering. For example T5 is an encoder-decoder model

A key feature of Transformer models is that they are built with special layers called attention layers. Attention consists in focusing on specific part of the sequence.

If you look at the picture of the transformer above, you can several Multi-head attention blocks. These blocks implement the attention mechanism

![[attention.png]]

Ajouter une explication sur The

More details here: https://huggingface.co/learn/nlp-course

## High level overview of training

Let's consider how the training occurs for a single sentence in the context of translation from english to french. The goal is to predict next output based on inputs and previous outputs​

```python
input_seq = "The delighted dog"
output_seq = "Le chien très heureux"
```

```
Input:
[The, Delighted, Dog]          output
						=>   	1 * V * 4 array
Target shift right              [w1,w2,w3,w4]
[<s>, Le, chien, très]
```

Here `wi` is an array of size `1*V` (V is the vocabulary size) where each element is the probability that the corresponding word in the vocabulary is the target word

We can compare each `wi` to the corresponding target word to calculate the loss. No need to pass the words sequentially. ​

Even if the prediction for a word is bad, it has no impact on the prediction for the next word (teacher forcing)

### Tokenization

More details here: [[Tokenizer]]
Before everything, sequences are tokenized, which means each token (word or portion of word) of the input is mapped to a number based on a given vocabulary. Vocabularies are created prior to training. TOKENIZATION STEP IS NOT UPDATED DURING TRAINING

What our transformer will take as a sequence in the schema will look like this

```python
# numbers will depend on the vocabulary
# note that some tokenizers also add extra tokens
# before and after but we don't show them here
input = [45, 79, 87]
output = [21, 46, 76, 47]
```

To simplify, we will represent them as shown below

```python
# here <The> == 45
input = [<The>, <delighted>, <dog>]
output = [<Le>, <chien>, <très>, <heureux>]
```

Note that the transformer uses Outputs (shift right). It corresponds to

```python
# <s> is a start of sequence token
output_shift_right = [<s>, <Le>, <chien>, <très>]
```

### Embedding

More details here: [[Word Embeddings]]

This process is similar for `input` and `output_shift_right`.
Each of our token is converted to a vector of a large dimension, for ex 768. The embedding is supposed to capture the semantic and syntactic relations between words.
The training process will refine the embedding by updating the weight of the embedding layer

For our example we will use an embedding of dimension 4

```python
# numbers will change during training
embedded_input = [
				  [0.01 0.51 0.87 0.45], # embedding of <The>
				  [0.51 0.65 0.95 0.02],
				  [0.37 0.99 0.01 0.08],
				  ]
```

### Positional encoding

Because we pass the full sequence at once, we need to provide information about the position of the words. This is done by adding a vector directly to the embedded_input
For ex we can use a positional encoding that works as following:

$\text{PE}(pos, 2i) = \sin \left(\frac{pos}{10000^{2i/d}} \right)$

$\text{PE}(pos, 2i+1) = \cos \left( \frac{pos}{10000^{2i/d}} \right)$

Here d is the embedding dim (4), pos is the position of the token in the sentence, and 2i (resp 2i+1) is the position within the encoding
Here is the final positional encoding vector:

```python
# positions are 0-indexed
# here 0.54 corresponds to pos == 1 and 2i+1 == 1
# so we calculate cos(1/10000^(0)) = cos(1) = 0.54
positional_encoding = [
					   [0.00, 1.00, 0.00, 1.00],​
					   [0.84, 0.54, 0.01, 0.99],​
					   [0.91, -0.42, 0.02, 0.99],
					   ]
```

Then we sum of `embedded_input` and `positional_encoding`

In conclusion, the vector that enters the encoder and the decoder include information about the words through their embedding and their position through the positional encoding

### Individual components of the encoder and decoder

- Nx represents the fact that the layer is stacked a certain number of times, typically 6 - 96 layers.

- Multihead attention blocks compute a linear combination of the input sequence, allowing the model to focus on specific parts of the input sequence (more details after)

- The Feed forward layer introduce non linearity in the network to allow it to learn complex relationships. It typically consists of two linear transformations separated by a non-linear activation function, such as the ReLU. More details here: [[Activation function]]

- Add and norm layers work the same for Multihead attention and feed forward layers
  - The "Add" step involves adding the input of the transformer block to its output. This creates Residual connections, also known as skip connections, and they address the issue of vanishing gradients and facilitate training stability. See [[Vanishing gradient]]
  - The "Norm" step applies layer normalization so that the activations have a mean of zero and a standard deviation of one. It is applied independently to each token in the input sequence

### Role of attention blocks

In the encoder, the attention block takes the input sequence and returns a vector where each element is a linear combination of the inputs. This can be understood as if the model focuses on different part of the input when analyzing a given word instead of focusing solely on the word.

For the decoder, we take the outputs shifted right. That is because we want to predict a word based on the previous words. The attention works the same as in the encoder but in the returned vector, each element is a linear combination of the PREVIOUS elements. This allows the decoder to attend to relevant parts of the previous output sequence as it generates each token.

In the final attention block, all the information from the input sequence is used, but only the information from the previous elements is used for generating the output sequence. This ensures that the model leverages the context from the entire input sequence while generating each token in the output sequence.

### Final layer

The final layer consists in a dense layer followed by a softmax. Which means in the output, each element is a vector of size **vocab** with an associated probability (here vocab is the length of the french vocabulary).

Note: because we passed the output_shift_right, even if we made a mistake to predict a word, it won't have an impact on the prediction of the next word during training. This is called teacher forcing

```python
# numbers will change during training
# here the most probably token has a proba of
# 0.61. It is in position 21 which corresponds to the
# word Le
final_output = [
				  [0.01 .. 0.61 .. 0.2 ..],
				  [...],
				  [...],
				  ]
```

We use the `final_output` to calculate our loss and backpropagate the error. The typical loss function used is the cross-entropy loss, which measures the dissimilarity between the predicted probability distribution and the true probability distribution over the vocabulary.

### Bonus: Attention mechanism details

#### Key, Query, Value

How does the attention block actually work?

After the embedding, the vector is passed through 3 dense layers in parallels: the key, the query and the value.

**Key** (K): The key is a representation of all input tokens (or elements) in the sequence.

**Query**(Q): The query is a representation of the current input token (or element) for which attention is being computed and is used to determine which parts of the input sequence are most relevant or important for the current token.

**Value**(V): The value is another representation of all input tokens (or elements) in the sequence. It provides the actual information content associated with each token and is used to compute the weighted sum in the attention mechanism.

The dimension is retained so if L is the number of words in the sequence and D is the embedding dimension, key, query and values are of dimension `L*D`.

Attention score (AS) is calculated based on the query and the key. Each row is the attention vector associated with the corresponding word in Q. - Each attention vector is a linear combination of all the vectors in Q​

Then a scalar product with the value give the final vector. As the value is a linear transformation of the input, the final vector is a linear combination of the initial input.

$$
\begin{align*}
AS &= Softmax\left(\frac{Q \cdot K^T}{\sqrt{D}}\right) \\
O &= AS \cdot V
\end{align*}
$$

Note: In the decoder, we use Causal attention (no impact of next words)=> ${Q \cdot K^T}$ is replaced with ${Q \cdot K^T + M}$ where M is a mask matrix: upper triangle (non including diagonal is set to $-\infty$)

In the encoder-decoder attention, $Q$ comes from the output shifted right while $K$ and $V$ are from the input sequence.

![[Transformers- Attention calc.png]]

#### Multi head attention

In fact, instead of using a unique key, query and value, we use several heads. Each one will learn a different relationship between words.

The idea is to have different heads focusing on different part of the sequences before concatenating them and reshaping the output to be the same size as the original input.

![[Transformers - Multi Head attention.png]]

Let's consider a batch of `Bs` sequences, each containing `L` words (or less but we pad to the max sequence). The embedding dimension is `D` and the number of heads is `Nh`

- Q, K and V are `[Bs, L, D]`. ​
- Q, K, and V are linearly projected into multiple representations (heads) using separate linear layers. Each linear layer produces an output of shape `[Bs, L, Nh*D]`
- The outputs from the linear layers are reshaped to have the dimensions `[Bs, Nh, L, D]`. This reshaping separates the heads so that they don't interact with each other.
- Attention is applied to each head separately, treating Nh like batches. This means that attention is computed independently for each head. After this step, we only have one vector of size `[Bs, Nh, L, D]` (in the previous step we had one for K,Q and V)
- After the attention calculation, the outputs from the different heads are reshaped and concatenated to form a single tensor of shape `[Bs, L, Nh*D]`.
- The concatenated tensor is passed through a final linear layer to produce the output of the multi-head attention mechanism. This linear layer projects the concatenated tensor back to the original embedding dimension, resulting in an output tensor of shape `[Bs, L, D]`.

## Inference

In inference, we don't have the output in advance. So we predict word by word.
![[Transformers - Inference.png]]
