#datascience

## Overview

Tokenization is the process of breaking down a sequence of text into smaller units (words, subwords, ...). Each of these units is assigned a number, typically through a vocabulary or dictionary maintained by the tokenizer.

Tokenization is essential in NLP tasks because machines only understand numbers

```python
# before tokenization
input_seq = "The delighted dog"

# after tokenization
# numbers will depend on the vocabulary
# note that some tokenizers also add extra tokens
# before and after but we don't show them here
input = [45, 79, 87]
```

Depending on the tokenizer, the same word can be mapped to a different number.

## Application with HuggingFace

### Standard Usage

HuggingFace makes using the correct tokenizer easy. Each checkpoint (pretrained model) they provide has an associated tokenizer.

```python
# AutoTokenizer and AutoModel choose the correct model architecture based on
# checkpoint (pretrained model)
# https://huggingface.co/docs/transformers/model_doc/auto
from transformers import AutoTokenizer, AutoModel


# Replace 'bert-base-uncased' with the name of the model you want to use
model_name = "bert-base-uncased"

# Download pre-trained model/tokenizer
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModel.from_pretrained(model_name)
```

Below is an example of how to use it

```python
tokenizer(
    ["my text", "my long sentence"],
    return_tensors="np", # type of tensor to return, pytorch (pt), numpy (np)...
    padding="longest",  # Padding to the longest sequence in the batch
    truncation=True,  # Truncate sequences if they exceed max length
)
```

It returns

```python
{
    "input_ids": array([[101, 2026, 3793, 102, 0], [101, 2026, 2146, 6251, 102]]),
    "token_type_ids": array([[0, 0, 0, 0, 0], [0, 0, 0, 0, 0]]),
    "attention_mask": array([[1, 1, 1, 1, 0], [1, 1, 1, 1, 1]]),
}
```

- The special tokens `[CLS]` (101) and `[SEP]` (102) indicate the beginning and end of the sequence, respectively.
- Check the token identifier with `vocab`. For ex `tokenizer.vocab["my"]` returns `2026`
- For `"my text"`, last token is 0 and in the attention mask, last value is also 0, which means it should be ignored. This is because we padded to the longest sequence.

Note: While transformers can work with input sequences of variable length, Padding ensures that all sequences in a batch have the same length, facilitating efficient batch processing and parallelization.

Note 2: It is very easy to use the tokenizer output in the model

```python
batch = ["my sentence", "my other sentence"]
# here kwargs is all the keyword arguments you want to add
tokenized_batch = tokenizer(batch, **kwargs)
output = model(**tokenized_batch)
```

## Technical stuff on token_type_ids

BertTokenizer output has a field called `token_type_ids`

This field becomes relevant when working with models designed to handle multiple segments or sequences simultaneously.

**Single String or List of Strings:**

If you pass a single string or a list of strings to the tokenizer,
  the token_type_ids will be set to all zeros (0).
  This indicates that all tokens belong to the same segment or sequence.
  This will be the case in most problems. In this case, it can be safely ignored

```python
tokenizer("Single string", return_tensors='np')
# 'token_type_ids': array([[0, 0, 0, ..., 0]])


tokenizer(["String1", "String2"], return_tensors='np')
# 'token_type_ids': array([[0, 0, 0, ..., 0]])
```

**Several Strings:**

If you pass two strings to the tokenizer, the second string is considered part of another sequence.

In this case, the token_type_ids will be set to [0, 0, ..., 1, 1, ...] to distinguish between the two segments.

```python
tokenizer("String1", "String2", return_tensors='np')
# 'token_type_ids': array([[0, 0, ..., 0, 1, 1, ..., 1]])
```

Note that BERT tokenizer only allows for 2 separate sequences. This is due to how BERT is trained. When you tokenize a sequence, it adds a CLS token first and a SEP token last. If you have two sequences, the second sequence is added without CLS token so you have something that looks like `[<CLS>, <S11>, <S12>, .... <SEP>, <S21>, <S22>, ... <SEP>]`.
When BERT is initially trained, the CLS token predicts if the second sentence follows the first or not and SEP is there to show the separation between sentences.

**List of List of Strings:**

If you pass a list of lists of strings, the token_type_ids will reflect the placement of each list in the outer list. Each inner list is considered a separate sequence, and the corresponding token_type_ids will indicate the segment to which each token belongs.

```python
tokenizer([["String1", "String2"]], return_tensors='np')
# 'token_type_ids': array([[0, 0, ..., 0, 1, 1, ..., 1]])

```
