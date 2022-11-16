---
title: Perceiver IO
author: Samridhi Maheshwari
date: 2022-11-16
tag:
 - Transformers, Deep Learning, NLP
---

# Introduction

By processing high-dimensional inputs from modalities as diverse as vision, audio, touch, proprioception, etc., humans are able to perceive and comprehend their surroundings. However, the majority of machine learning models use modality-specific architectures and work with a predefined set of inputs and outputs that are connected to a single task. Every time inputs change, this forces researchers to redesign their architectures.

Designed to support a variety of inputs sizes, the recently proposed Perceiver architecture (Andrew Jaegle, 2021) produces excellent performance on domains like image, audio, point clouds, etc while scaling linearly in computation and memory with input size. However, this model can only generate simple outputs such as class scores.

To broaden the Perceiver model, the team responsible for creating Perceiver has proposed Perceiver IO, a general purpose neural network architecture that can easily integrate and transform arbitrary information for arbitrary tasks. Perceiver IO maintains Perceiver’s appealing properties; scaling linearly with both input and output sizes and achieves outstanding results on tasks with highly structured output spaces, such as language and visual understanding.

# Transformers aren’t all you need

Take language translation as an example, which is a frequent application for the Transformer architecture. Transformer performs well in general because it can handle lengthy input sequences concurrently, which is useful when we have to refer to other words for context (Neural Machine Translation, Word Prediction, etc). Every word in the input sequence is mapped to every other word in a transformer architecture to gain a richer knowledge of the semantic context and carry out holistic learning. In a general Transformer architecture, the first input data goes through a multi-headed attention layer which then goes through a feed-forward layer. Inside the multi-headed attention module there are multiple self-attention layers. 

![Untitled](CS595J%20Seminar%20Blog%20-%20Perceiver%20IO%2013300deec0184fda91b564a47f46f80d/Untitled.png)

Inside the self-attention module, we feed the input to three distinct fully connected layers to create query, key and value vectors. The queries and keys undergo dot product matrix multiplication to produce a scoring matrix. The score matrix determines how much focus should a word be put on other words. Then a soft max layer is added to get the highest probability values and this helps our model to provide more confidence on which words to attend to. And then it gets multiplied to value vector to get output vector. Because of the dot product matrix multiplication between Query and Key vectors, the complexity of transformers instantly shoots upwards quadratically in space as input increases. 

For language tasks, then max length of input sequences would be batched to around 1000 or so which can be handled given the strength of today’s hardware. However, when considering images, input sequences would be all the pixels in the image**.** For a simple image of 256 x 256, the complexity of input sequence is 65536 x 65536 which is slow and hard to handle computationally. 
.

![Untitled](CS595J%20Seminar%20Blog%20-%20Perceiver%20IO%2013300deec0184fda91b564a47f46f80d/Untitled%201.png)

Several approaches have come up to provide alternative solutions such as VIT Transformers where images have been broken into batches and then fed into the network, however these architectures don’t solve the quadratic complexity issue. 

# Original Perceiver

The Perceiver architecture tries to reduce this space complexity to a limit such that it does not become quadratic. The authors in the Perceiver paper added a cross attention layer between the input sequence and multi-headed attention . 

![Untitled](CS595J%20Seminar%20Blog%20-%20Perceiver%20IO%2013300deec0184fda91b564a47f46f80d/Untitled%202.png)

In attention module, where we perform matrix multiplication between Query and Key vectors where both vectors were of size m x m (length of input sequence), in cross attention, the query vector is of size n where n < m. Using this new length, our space complexity reduces to m x n. This query vector of size n is called the latent array. A latent array acts as a filter of inflow of data into the multi-headed attention. The latent vector queries only a few input sequences from all of the input sequences, however, in the next step the input sequence is being queried again by the latent array. Thus, it acts similar to an RNN architecture where important features are moved to the next steps and weights are shared amongst them. Because of this functioning of the latent array, this architecture can work with any kind of input sequences, from text to image to videos even to word cloud.

# Perceiver IO

Rather than output a single category, Perceiver IO aims to have the same level of generality with respect to its outputs as the Perceiver has with respect to its inputs. It produces arbitrary sized output arrays. We can predict each element of the output array using another attention module by querying the same latent array used in the encoder module. This cross attention processing uses a query feature vector unique to the desired output element. In other words, a query array is defined with the same number of elements as the desired output. The queries may be hand-designed, learned embeddings, or a simple function of the input. They attend to the latents to yield an output array of the desired shape.

![Untitled](CS595J%20Seminar%20Blog%20-%20Perceiver%20IO%2013300deec0184fda91b564a47f46f80d/Untitled%203.png)

With this change, both the encoder and decoder take in two input vectors, the first used as input to the module’s key and value neural networks, and the second used as input to the module’s query network. The module’s output has the same dimension as the query input vector. This allows the decoder modules to produce outputs with different sizes.

The Perceiver IO pipeline comprises three main steps: 

1.  Input arrays are encoded to a latent space
2.  The latent representation undergoes processing multiple times (the modules take in and return arrays in the latent representation each time). The architecture components are implemented using Transformer style attention modules. Each module applies a Query, Key, Value attention operation followed by a feed forward neural network. This processing allows the latent representation to become more refined. 
3. The latent vector is decoded to produce outputs

To capture the structure of the output space, it is made sure that this query vector contains appropriate information. Byte information in the query vector should reflect the output of the task in hand, and ideally capture the structure needed in the outputs. This may include the spatial position in an image or the position of an output word in a sequence. Each output point depends only on its query and the latent array, allowing decodes of outputs in parallel. This property allows amortization of model training on datasets of very large output size.

# Results

To evaluate the generality of Perceiver IO, the DeepMind team conducted experiments on various domains, including language tasks, audio visual encoding, symbolic representations for games - StarCraft II, optical flow, etc.

![Untitled](CS595J%20Seminar%20Blog%20-%20Perceiver%20IO%2013300deec0184fda91b564a47f46f80d/Untitled%204.png)

![Untitled](CS595J%20Seminar%20Blog%20-%20Perceiver%20IO%2013300deec0184fda91b564a47f46f80d/Untitled%205.png)

![Untitled](CS595J%20Seminar%20Blog%20-%20Perceiver%20IO%2013300deec0184fda91b564a47f46f80d/Untitled%206.png)

In the results, the Perceiver IO achieved state of the art results on tasks with highly structured output spaces, matching a transformer-based BERT baseline on the GLUE language benchmark without the need for input tokenization. It also achieved state of the art results on optical flow estimation. These results indicate its promising future as a general-purpose neural network architecture.

![Untitled](CS595J%20Seminar%20Blog%20-%20Perceiver%20IO%2013300deec0184fda91b564a47f46f80d/Untitled%207.png)

# Conclusion

In conclusion, Perceiver IO is an architecture capable of handling general purpose inputs and outputs while scaling linearly in both input and output sizes. This architecture achieves good results in a wide variety of settings, making it a promising candidate for general purpose neural network architecture. Perceiver IO leverages the expressive power of latent attention and uses learned queries to expose a simple and unified interface that can handle multimodal and multitask settings.

## Reference 
- Andrew Jaegal et al. Perceiver. PMLR 139, 2021. 
