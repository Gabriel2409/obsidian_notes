#datascience

## Vector space

Vector space models allow you to represent words and documents as vectors and capture relative dependencies between words:

- Syntaxic dependencies​
- Semantic dependencies

The larger the dimensionality of the vector space, the more subtle relations you can capture between words. However, as the dimensionality increases, so does the computational cost of training and inference.
For example, with BERT, we use an embedding of size 768

Once the vector space is created, cosine similarity will tell you how similar the words are
![[Word Embeddings cosine similarity]]

- cake and sweet are quite similar: `θ≈0 => cos(θ)≈1`
- cake and truth are very different: `θ≈90° => cos(θ)≈0`
- Note: antonyms have a cosine similar of -1

## Create word embeddings

To create word embeddings you always need two things. A corpus of text and a training task.

The corpus of text is the collection of text data that will be used to train the word embeddings. It could be a large dataset of sentences, paragraphs, or even entire documents.

In the context of word embeddings, the training task typically involves predicting context or nearby words given a target word.

During training, the tokenized input (words or subwords) is passed through an embedding layer. which maps each token to a vector representation (word embedding). The weights of the embedding layer are updated along with the weights of other layers in the model based on the loss during backpropagation.

After the task is complete, you now have an embedding layer that converts token into word embeddings. This layer can be frozen and used for new tasks.

## Application

It is possible to add an embedding layer with pytorch:

```python
from torch.nn import Embedding
```

However in practice, you will often use pretrained model so you won't have to modify the embedding layer.

```python
from transformers import AutoTokenizer, AutoModel

model_name = "bert-base-uncased"

tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModel.from_pretrained(model_name)

print(model.embeddings.word_embeddings)
# returns Embedding(30522, 768, padding_idx=0)

print(len(tokenizer.vocab))
# returns 30522
```

Note: The way it works it the following

- First, get a token, for ex "The"
- Pass it through the encoder, you get a number, for ex 45
- To get the embedding, you read the row 45 of the embedding matrix
- It is the same as if we had one hot encoded "The" during tokenization and then done a scalar product with the embedding matrix. However, doing a look up of a row is more computationally efficient and this is why Embedding layers are basically look up tables

## Word embedding with Word2Vec

One of the first word embeddings was created with Word2Vec
The idea is to predict a word based on the surrounding words in a sentence of the corpus​.
It is a self supervised task because input data is unlabeled​: all the necessary context is provided by the adjacent words​

### Skipgram vs CBOW

- Continuous bag of words : The objective of the model is to learn to predict a missing word given the surrounding words.​

```bash

W(t-2)
W(t-1)
		AVERAGE -> Predict W(t)
W(t+1)
W(t+2)
```

- Continuous skip-gram, also known as the skip-gram with negative sampling, which does the reverse of the continuous bag of words method : The model learns to predict the word surrounding a given input word.​

```
		PREDICT W(t-2)
		PREDICT W(t-1)
W(t)  ->
		PREDICT W(t+1)
		PREDICT W(t+2)
```

In the next part, we focus on CBOW

### Context window parameter

C represents the number of words we look at on each side of the word we want to predict. It is a hyperparameter. ​

The context window for each word can be the same (size C) or a random int between 1 and C (in this case, C is the max context half size: more variability during training)​

Example: The cake is a lie: If the center word is "is" and C = 1, then we also look at the words "cake" and "a"

### Inputs and outputs

For the tokenization, here, we use one hot encoding. So if a word is associated to token 170 in a vocabulary of size 35000, it will be a vector with ony 0 except in position 170 where it is a one.

The input of the model will be the average of all the words in the context window.
The output will be the center word

![[Word Embeddings - inputs and outputs of word2vec.png]]
