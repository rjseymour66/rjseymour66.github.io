---
title: "Large Language Models"
linkTitle: "LLMs"
# weight: 1000
# description: 
---

- Deep neural network models
  - mimics the human brain--these are layers of interconnected nodes that process data and learn patterns through training
  - 3 or more layers
- Before, could only do pattern recognition
- Now, they understand complex tasks and can generate text
- Trained on large quantities of text data that lets them capture deeper contextual info and subtleties in human language
  - improved translation
  - sentiment analysis
  - question answering
- More broadly usable than previous NLP
- Transformer architecture lets them capture nuances, context, and patterns of language
  - transformer architecture and large data sets

## What is an LLM

- LLM is a deep neural network that understands and generates human-like text after they are trained on large amounts of text data
- Large is size of training text and parameters
  - billions of parameters - adjustable weights in the network that are optimized during training to predict the next word in a sequence
  - this is called next-word prediction
- transformer: pays selective attention to different parts of the input when making predictions
- Called generative AI because they can generate text

The algorithms that implement AI are the focus of ML:

- make predictions or decisions based on data without being explicitly programmed

Traditional ML vs Deep Learning

- Trad ML needs human experts to provide the most relevant examples for the model
- Deep Learning doesn't require experts provide examples

## Applications of LLMs

Bc they can parse and understand unstructured data, they can be used:

- content creation
- fiction writing
- computer code
- chatbots
- effective knowledge retrieval in legal or medical docs
  - summary
  - answering technical questions

## Stages of building LLMs

- pre-training: initial phase where you train the model on a large and diverse data set
  - can also fine-tune existing OOS LLMs for domain specific tasks
  - PyTorch deep learning library
- domain-specific or custom-built LLMs outperform general ones like ChatGPT
  - provides data privacy--you don't have to upload confidential info to ChatGPT-like program
  - smaller, so can deploy on customer devices to reduce latency and server-related costs
  - control updates and modificaitons

Main stages are pre-training and fine-tuning:

1. Pretrain on large diverse dataset to get an understanding of language.
   - Also called base or foundation model
   - Raw (unlabeled) text
   - maybe filter out special characters
   - uses self-supervised learning to generate its own labels form input data
   - few-shot capabilities: it can perform tasks based on a few examples
2. Fine-tuning: narrower, labeled dataset that is more specific to tasks or domains
   - two popular categories are instruction fine-tuning and classification fine-tuning
     - instruction fine-tuning: labeled dataset is instruction and answer pairs. Ex: question to translate a text and then the translated text
     - classification fine-tuning: texts and class labels

## Transformer architecture

> The architecture of large language models (LLMs) is primarily based on the transformer architecture, which utilizes an attention mechanism to process and generate text by selectively focusing on different parts of the input sequence.

Deep neural network architecture that has an encode and decorder:

- encoder: processes input text and encodes it into a series of numerical representations or vectors
  - vector: 1D tensor that represents text or image data as numbers
- decoder: takes encoded vectors and generates output text

They are connected by a self-attention mechanism

- weigh the importance of different words or tokens as relevant to each other
  - computes a context vector from each input element by combining all other input elements and using attention weights to give each element importance
- BERT is an example (Bidirectional Encoder Representations from Transformers)

BERT vs ChatGPT:

- BERT: built on the encoder
  - masked word prediction - predicts masked or hidden words in a sentence
  - good for text classification tasks like sentiment prediction and document categorization
- ChatGPT: built on decoder
  - generates texts
  - good for machine translation, text summarization, fiction writing, writing computer code, etc
  - good for zero-shot and few-shot learning
  - zero-shot: ability to generalize to completely unseen tasks without prior specific examples
  - few-shot: learning from minimal number of examples

"Transformers" and "LLM" are often used interchangeably, but there are some differences:

- not all transformers are LLMs, bc transformers can be used for computer vision
- not all LLMs are transformers -- there are new and different architectures

## Large datasets

Pretraining datasets consist of billions of words:

- These foundational models make them great for downstream training tasks
- requires significant resources and is very $$$
- There are many pretrained LLMs that are available as OSS models
  - you can fine tune with much smaller datasets

## Closer look at GPT

[Generative Pre-Training](https://cdn.openai.com/research-covers/language-unsupervised/language_understanding_paper.pdf).

GPT was pretrained on next-word prediction task that predicts the next word in a sentence by looking at the words that come before it

- this is self-supervised learning, which is self-labeling
- you don't need to collect labels for the training data, you can use the structure of the data itself
- unlike transformer architecture, this only has a decoder, no encoder
  - left to right processing
  - autoregressive model -- they predict text one word at a time
  - these models incorporate their previous outputs as inputs for future prediction
  - each word is chosen using the sequence of words that precede it

GPT-3 is much larger than original transformer model:

- has 96 transformer layers
- 175B parameters

Emergent behavior:

- when a model performs tasks that it wasn't explicitly trained on
- happens as a byproduct of the model's exposure to lots of data