---
layout: article
title: NLP
comments: true
categories: data_science
image:
  teaser: jupyter-main-logo.svg
---

## Representation
- Natural language is the means of communication for humans. It represents concepts, entities, and the relationships among them in the real word and therefore is quite complex. However, on the representation level it consists of discrete units of letters, words, and sentences. 

-  Mathematically speaking, a language model is the joint probability distribution of words such that correct and meaningful sequence of words/characters have a high probability distribution while incorrect sequences have low probability. This joint distribution can be factorized further based on assumptions like n-gram models meaning that only n sequences depend on each other and not more. 

- Word2Vec models: a class of NN models that learn a vector representation for words from an unlabeled corpus of text. 
    + Skip-Gram: Based on the notion that words appearing in similar contexts are related to each other semantically. Skip-gram predicts a window of neighboring words from a single word.
        * We convert the corpus into samples with a neighboring window of n-grams on both sides of a word.
        * input is one-hot representation of a single words. each neighbor word has a corresponding output. Therefore, outputs are n words in the neighborhood (both sides), each having a one-hot representation.
        * The model has a bottleneck hidden layer between the input word and output neighboring words. The weights between one-hot input and hidden layer will be used as m-dimensional word embedding matrix (W) after training. 
        * no nonlinearity is used in the bottleneck layer since inputs are one-hot and only activate a single row of the weight matrix.
        * the weight matrix between the bottleneck layer and each output node is shared.
        *  a softmax is used for each neighboring word output to produce a one-hot vector for that neighboring word.
            -  In the two-class logistic regression, we pass the single output through a sigmoid function, $$\frac{1}{1+exp(-x)}$$, to crush it into the $$[0,1]$$ range and interpret it as a probability i.e. $$P(Y=0)= sigmoid(x), P(Y=1)=1-P(Y=0)$$. 
            -  In the multi-class logistic regression, with have $$K$$ outputs so we can't just say one minus probability of one class. Therefore, we need a generalization of the sigmoid to crush all the outputs into the $$[0,1]$$ range to interpret them as probabilities. If we write the output equations for the other class in the two class logistic regression, we can [derive the generalization](https://stats.stackexchange.com/questions/233658/softmax-vs-sigmoid-function-in-logistic-classifier) to softmax function $$P(Y=j)=\frac{exp(z_j)}{\sum_k exp(z_k)}$$.
        * The loss function is negative log-likelihood loss i.e. $$L= -\frac{1}{n} \sum_k \log(a)$$
            - In two class logistic regression, we can use cross-entropy loss with the sigmoid output since there are only two outputs i.e. $$L= - \frac{1}{n} \sum_x (y \log(o) + (1-y)\log(1-o))$$

            - In the multiclass logistic regression, the equivalent of cross-entropy for softmax output is negative log-likelihood loss i.e. $$L= - \frac{1}{n} \sum_x \log(o_y)$$.

        * Training Algo: 
            - Represent each word as a d dimensional vector (or connect one-hot word input to a d-dim bottleneck so that weight matrix (W) becomes a d-dim word representation).
            - Represent each context word as a d dimensional vector (or connect the d-dim bottleneck layer to n one-hot outputs for contexts).
            - Initalize all vectors to random weights.
            - Arrange vectors in two matrices, W and C.
            - feed in the corpus formatted as bi-ngrams. 
            - Negative sampling objective is based on NCE which basically constructs a fake dataset of bi-ngrams from the original bi-ngrams by replacing the input word with a random one while keeping the context the same. Objective then tries to assign high probability to correct bi-ngrams while assigning low probability to fake bi-ngrams. 
            - After training, Throw away the C matrix and use the W matrix as word-embeddings. 
        * If we don't throw away the C matrix, the product $$W.C$$ will produce the co-occurrence frequency matrix for each word and its ngrams. Therefore, word2vec is essentially doing a matrix factorization on that matrix in a more algorithmically efficient way than LSA. Even LDA can also be interpreted as a matrix factorization.
       
    + Continuous BoW (CBoW): It's the exact mirror of skip-gram model where a word is predicted from it's neighbors instead. We train a simple one-layer logistic regression model to predict the one-hot encoding of a word from the one-hot representation of its neighbors. 
        * After the training with maximum likelihood, the weight matrix can be used as a word embeddings. 
        * typically used as a pre-training for initialization of the word-vectors before learning an embedding. 
    + word2Vec encoding: Based on the notion that words appearing in similar contexts are related to each other semantically. We learn word vector representations by defining a model that predicts a word given its context and context given a word. We condition a word on its neighbors. The probability of a word (center) given its neighbors (n) is determined by the normalized softmax of the distance between two word vectors (dot product distance for skip-gram) $$p(n|c)=\frac{\exp{u_n^T . u_c}}{\sum_c \exp{u_n^T . u_c}}$$ (i.e. angle between the vectors). 
        * We let the word embeddings be parameters. Training an LSTM with softmax to predict context from a word learns word2vec representations.
        * After learning the word embeddings, word vectors will be stored in a lookup table with an index for each word.
        * The similarity of two words is represented by the angle between their vectors (cosine similarity). Therefore, similar words will be parallel and very dissimilar words will be orthogonal. In one-hot all words are orthogonal which doesn't make sense given their semantic similarities.
        * the result is a dense vector representation of words that embeds words appearing in each others context in the same region of the vector space (low distance).
        * There usually is a linear relationship between word vectors for example relationship between king/queen is similar to man/woman
    + Glove: goes from word-counts (bag-of-words) to nice meaningful embedding properties of the word2vec model. The crucial insight is that co-occurrence probabilities of words might be volatile but the ratio of co-occurrence of words is a much more stable measure for embedding meaning. 
        * the way this ratio is mapped to word vectors is equating the probability with distance $$w_i.w_j=\log p(i|j)$$.
        * can we use density ratio estimation here?
    + Skip Thought Vectors: Generate sentence codes in the style of word embeddings to predict context sentences. One encoder and two decoder to predict previous and next sentences.
        * sentence_t -> sentence_{t-1} sentence_{t+1} [Kiros et al.,2015]
    + Paragraph vector: A paragraph is represented as a vector in the same space as the single-word embeddings.
        * non-RNN model. A paragraph matrix D is a bag-of-sentences representation and a vocabulary matrix W is a bag-of-words representation. we train on the bag-of-words W to predict next words using bag-of-sentences D as context.

- LDA: While word2vec tries to predict a word from its local context, LDA tries to predict the word from the global document (LDA generative model first chooses a topic, then samples a word from that topic). While word2vec representations are dense, LDA representations are sparse (only a few topics).

- Obviously the number of words in a language is much smaller than the combination of the letters in that language. Why not learn a language-specific word manifold that maps characters to words? The manifold represents the language and only combinations of characters on the manifold would be allowed.
- Then we can use the char-level language models instead of word-level models and reduce the dimensionality of the problem since number of chars are multiple orders of magnitude smaller than words. 

- a measure: can we take these word vectors and use them in other task like language modeling, translation, etc


### NLP tasks
Easy
• Spell Checking
• Keyword Search
• Finding Synonyms
Medium
• Parsing information from websites, documents, etc.
Hard
• Machine Translation (e.g. Translate Chinese text to English)
• Semantic Analysis (What is the meaning of query statement?)
• Co-reference (e.g. What does "he" or "it" refer to given a document?)
• Question Answering (e.g. Answering Jeopardy questions)
• Summarization
• Text Generation



## Generating sentences from a continuous space

- Auto-encoders: Typically composed of two RNNs, The first RNN encodes a sentence into an intermediate vector, The second RNN decodes the intermediate representation back into a sentence, ideally the same as the input.
    + Regular auto-encoders learn only discrete mappings from point to point spanning the whole lower dimensional space. However, if we want to learn holistic information about the structure of sentences, we need to be able to fill sentence space better as a lower dimensional manifold thus VAEs are better. 
    + In a VAE, we replace the hidden vector z with a posterior probability distribution q(z|x) conditioned on the input, and sample our latent z from that distribution at each step. We ensure that this distribution has a tractable form by enforcing its similarity to a defined prior distribution, typically some form of Gaussian. Alternatively we can use an implicit distribution which is much more flexible and can better model the real posterior. 
    + Optimization problems: Decoder too strong, without any limitations just doesn’t use z at all – Fix: KL annealing – Fix: word dropout
        * Word dropout – Keep rate too low: sentence structure suffers – Keep rate too high: no creativity, stifles the variation
    + Used VAE to create language models on the Penn Tree-bank dataset, with RNNLM as baseline
        * Task: train an LM on the training set and have it designate the test set as highly probable. RNNLM outperformed the VAE in the traditional setting. However, when handicaps were imposed on both models (input-less decoder), the VAE was significantly better able to overcome them.
        * Task: infer missing words in a sentence given some known words (imputation).
            - Place the unknown words at the end of the sentence for the RNNLM –  RNNLM and VAE performed beam search (VAE decoding broken into three steps) to produce the most likely words to complete a sentence
            - Precise evaluation of these results is computationally difficult
            - Instead, create an adversarial classifier, trained to distinguish real sentences from generated sentences, and score the model on how well it fools the adversary
            - Adversarial error is defined as the gap between chance accuracy (50%) and the real accuracy of the adversary.ideally this error will be minimized
            - Several other experiments in the appendix showed the VAE to be applicable to a variety of tasks – Text classification – Paraphrase detection – Question classification
        * Task: sample from the VAE, interpolate between sentences by exploring the latent dimension.




https://www.analyticsvidhya.com/blog/2017/01/ultimate-guide-to-understand-implement-natural-language-processing-codes-in-python/



# Neo

## Motivation
Machine learning and data science in general are concerned with discovering patterns and insights in large collections of data. The advent of deep learning models have had a major impact on many task in computer vision and natural language processing as well as other domains. In essence, machine learning models condense the raw into a machine representation, from which insights are extracted and communicated to provide business and scientific value.

On the other hand, language is the primary facet of communication for humans. Communicating insights from a machine representation back into human language is critical for making the findings accessible to people. Natural language generation(NLG) is the task of automatically generating language from a machine representation of data e.g. a knowledge graph.


## Objective
The objective of this project to bridge the gap in communication of insights to humans using human language. In the specific case of the Apollo project, this can help analysts understand the information contained in the knowledge graph in the form of statements about the existing relationships in the graph. 


## conditional generation of summaries based on knowledge graph. 

Similar setting to labeled sequence transduction task, with a given input sequence and a specified condition, we generate an output sequence to reflect the content of input sequence that satisfies the condition. Examples:
-  using labels to moderate politeness in machine translation
-  modifying the output language of a machine translation system
-  controlling the length of a summary in summarization
-  morphological reinflection (combinig a root word with suffixes/prefix to make new syntactic and semantic variations). 



- This will be a layer on top of th Appolo project that connects it to humans and users. The knowledge graph extracts entities and events from many news articles and represents them in a graph. 
- The idea is to generate sentences and paragraphs conditioned on the knowledge graph. This will condense information from many news sources and many relationships into a human readable passage. 
- This can be used by analysts trying to make sense of an event. It will help them understand all aspects of the event by reading a single article without the need to read many articles about the topic. 
- Download and run this to show you can generate sentences [pytorch implementation](https://github.com/kefirski/pytorch_RVAE)
- model to be used is conditional VAE. 
- need to find a way to represent the knowlege graph to input to the CVAE. Maybe graph convolution? 

## Dataset
- Thomson-Reuters dataset and the associated knowledge graph from the Apollo project 
- example of relationship between text and the knowledge graph:
    + sentence [GM invests 100MM in Lyft] ->
        * knowledge graph [nodes(GM, Lyft), relationship(investing)]


## Methodology
- A conditional recurrent VAE [1-5] to reconstruct the input sentence conditioned on the  knowledge graph representation of the relationship. 

- Possible to use discrete latent variable models i.e. Concrete[], vector quantisation (VQ) [],REBAR [], RELAX []. 


## References
[1] Kingma, Diederik P., and Max Welling. "Auto-encoding variational Bayes." arXiv preprint arXiv:1312.6114 (2013).

[2] Kingma, Diederik P., et al. "Semi-supervised learning with deep generative models." Advances in Neural Information Processing Systems. 2014.

[3] Maaløe, Lars, et al. "Auxiliary deep generative models." arXiv preprint arXiv:1602.05473 (2016).




[4] Bowman, Samuel R., et al. "Generating sentences from a continuous space." arXiv preprint arXiv:1511.06349 (2015).

[5] Zhou, Chunting, and Graham Neubig. "Multi-space Variational Encoder-Decoders for Semi-supervised Labeled Sequence Transduction." arXiv preprint arXiv:1704.01691 (2017).

[6] Hu, Zhiting, et al. "Controllable Text Generation." arXiv preprint arXiv:1703.00955 (2017).

[7] Hsu, Wei-Ning, Yu Zhang, and James Glass. "Unsupervised Learning of Disentangled and Interpretable Representations from Sequential Data." Advances in neural information processing systems. 2017.

[8] Gupta, Ankush, et al. "A Deep Generative Framework for Paraphrase Generation." arXiv preprint arXiv:1709.05074 (2017).

[9] Guo, Jiaxian, et al. "Long Text Generation via Adversarial Training with Leaked Information." arXiv preprint arXiv:1709.08624 (2017).

[10] Ni, Jianmo, et al. "Estimating Reactions and Recommending Products with Generative Models of Reviews." Proceedings of the Eighth International Joint Conference on Natural Language Processing (Volume 1: Long Papers). Vol. 1. 2017.




[6] Jang, Eric, Shixiang Gu, and Ben Poole. "Categorical reparameterization with gumbel-softmax." arXiv preprint arXiv:1611.01144 (2016).

[7] van den Oord, Aaron, and Oriol Vinyals. "Neural Discrete Representation Learning." Advances in Neural Information Processing Systems. 2017.

[8] Tucker, George, et al. "REBAR: Low-variance, unbiased gradient estimates for discrete latent variable models." Advances in Neural Information Processing Systems. 2017.

[10] Grathwohl, Will, et al. "Backpropagation through the Void: Optimizing control variates for black-box gradient estimation." arXiv preprint arXiv:1711.00123 (2017).