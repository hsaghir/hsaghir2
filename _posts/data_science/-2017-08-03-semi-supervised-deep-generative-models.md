---
layout: article
title: The unreasonable elegance of deep generative models
comments: true
categories: data_science
image:
  teaser: jupyter-main-logo.svg
---

## Background

With the advent of [deep learning](), we've had some very exciting breakthroughs in machine learning. However, the caveat is that with the exception of a few [cool ideas](), almost all recent success stories of deep learning come from the supervised learning front. Cases where vast amount of labeled data is available.

Between the [Internet](), [smart cities](), [Internet of things](), [big data](), [Block chain](), etc it's safe to say data is [eating the world]() and we have no shortage of data. However, almost all of these data resources are unlabeled data. Obtaining labeled data is often very expensive, time-consuming and not scalable while unlabeled data is vast and cheap. So the question is, how can we learn from limited labeled data and can we use unlabeled data to aid our models? Some very promising approaches to answering these questions are [transfer learning](), [few-shot learning](), [active learning](), [reinforcement learning](), and [semi-supervised learning](). 

In this post, I will focus on the last one, a common real world situation where only a small subset of our datapoints are labeled while the rest are unlabeled. The question is, can we do something smarter than just ignoring the unlabeled data? The answer is yes and it's semi-supervised learning. The idea is to use our vast resources of unlabeled data to aid our models in a supervised learning task. Although unlabeled data might not be able to help in discriminative tasks, it can be useful in finding the right representation for the data. 


Some approaches that have been explored in the literature of semi-supervised learning are:

    + Self-training, where the model is fed the data it has confidently classified. The downside is that this approach can reinforce poor predictions.
        * e.g. Transductive SVM -> use distance to margin as confidence. The problem is that it's not really scalable.

    + Graph-based approaches: the idea is to propagate the label information from labeled nodes to unlabeled nodes (MAP inference?)

    + neural net based approaches: the idea is to train a classifier along with an autoencoder (or other unsupervised embeddings) as a regularizer. 

    + Manifold tangent classifier: the idea is to train contrastive autoencoders to learn the manifold on which the data lies. Followin that with TangentProp to train a classifier that is approximately invariant to local perturbations along the manifold. (Tangent prop has an additional derivative of weights penalty term for known invariances in the data)
    
    + Manifold learning using graph-based methods + SVM kernel (rbf) method.



## Representation learning with VAE
M1 is a simple VAE model for unsupervised feature learning. The learned features are used for training a separate classifier. Approximate samples from the posterior distribution over the latent variables p(z|x) are used as features to train a classifier that predicts class labels y from data in a lower dimensional space. 

<img src="/images/VAE_intuitions/vae_semi_M1.png" alt="semi-supervised model inference" width="350" height="350">

Let observations be $$X = {x_1, x_2, .., x_n}$$. We assume a simple generative model with a set of latent variables $$Z = {z_1, z_2,...,z_n}$$. 

$$ Z ~ p(Z) = N(0,I) ;  X ~ p(X|Z) = N(\mu, \sigma^2)$$

![alt text](/images/VAE_intuitions/vae_semi_M1.png "a simple vae latent variable model")

- Variational Inference turns the inference into optimization. It posits a variational family of distributions over the latent variables, and fit the variational parameters to be close (in KL sense or another divergence like BP, EP, etc) to the exact posterior. KL is intractable (only possible exactly if q is simple enough and compatible with the prior), so VI optimizes the evidence lower bound (ELBO) instead which is a lower bound on log p(x). Maximizing the ELBO is equivalent to minimizing the KL, note that the ELBO is not convex. The ELBO trades off two terms, the first term prefers q(.) to place its mass on the MAP estimate. The second term encourages q(.) to be diffuse.

- Note that in the above derivation of the ELBO, the first term is the entropy of the variational posterior and second term is log of joint distribution. However we usually write joint distribution as $$p(x,z)=p(x|z)p(z)$$ to rewrite the ELBO as $$ E_q[\log\ p(x|z)+KL(q(z|x)\ | \ p(z))]$$. This derivation is much closer to the typical machine learning literature in deep networks. The first term is log likelihood (i.e. reconstruction cost) while the second term is KL divergence between the prior and the posterior (i.e a regularization term that won't allow posterior to deviate much from the prior). Also note that if we only use the first term as our cost function, the learning with correspond to maximum likelihood learning that does not include regularization and might overfit.

- What we need for optimization is actually the gradient of the ELBO not the ELBO itself. Using pathwise gradient (reparameterization trick) we can calculate the gradient without evaluating the ELBO

## Conditional VAE

## Continuous+Categorical latent variables (M2)


## Semi-supervised learning with VAEs

- M2 is a probabilistic model that describes the data as being generated by a latent categorical class variable $$y$$ in addition to a continuous latent variable $$z$$. In this model, the VAE acts as a regularizer for the classifier. Therefore, The total ELBO will be the sum of the ELBO of the classifier and the ELBO of the VAE regularizer. Depending on whether the label is present or not for an instance, the total ELBO will have two forms which are also summed to form the semi-supervised total ELBO.

<img src="/images/VAE_intuitions/vae_semi_M2.png" alt="semi-supervised model inference" width="350" height="350">

- In a traditional classifier we predict labels, $$y$$, from data, $$x$$, i.e. $$p(y|x)$$. In M2 semi-supervised model, the label $$y$$ is considered to be a categorical latent variable since for some data points the label is known and for other points it's unknown. The model has other latent variables $$Z$$. If we can calculate the joint distribution $$p(X,Y,Z)$$, we can then calculate the conditional of label given data i.e. $$p(y|x)$$ as a semi-supervised classifier. 

<img src="/images/VAE_intuitions/semi_sup_classifier.png" alt="traditional vs semi-supervised classifier" width="350" height="350">

- The inference model for the semi-supervised model is as follows: $$q(Y|X) = categorical(\lambda = DNN_\phi(X)), q(Z|X,Y)= N(\mu = DNN_\phi(X), diag(\sigma^2 = DNN_\phi(X)))$$ where the parameters $$\lambda, \mu, \sigma$$ are parameterized by the encoder deep neural network. Softmax function can be used as the categorical distribution. 

- The priors for the generative model for the semi-supervised model are: $$Z ~ p(Z) = N(0,I) ; Y ~ \frac{1}{number of labels} ; X ~ p(X|Y,Z) = DNN(X; Y,Z,\theta)$$

- So there are actually three neural nets on the encoder side. One DNN for the $$\lambda$$ parameter of the categorical latent variable (label). One for $$\mu$$ and one for $$\sigma$$.
    + First we use X to determine the $$\lambda$$ of categorical variable using first encoder network.
    + second we sample the label categorical $$Y$$
    + third we use both $$X, Y$$ to determine the $$\mu, \sigma$$ of $$z$$ using the second and third encoder networks.
    + fourth we sample $$Z$$ from the normal distribution
    + fifth we generate $$X$$ from $$Y, Z$$ using the decoder network.

### Stacking a VAE and a SS-VAE (ELBO derivation)

(M1+M2) This model stacks M1 and M2 models. First we learn features  unsupervised in M1 and then we feed M1 learned features to M2 and learn a semi-supervised model.

<img src="/images/VAE_intuitions/vae_semi_M1_M2.png" alt="semi-supervised model inference" width="350" height="350">

- The joint probability distribution of the model is $$p(X,Y,Z1,Z2) = p(Z2)p(Y)p(Z1|Z2,Y)p(X|Z1)$$. Perform inference on latent variables in this model we use variational inference. 
- First step in variational inference is throwing a variational family of distributions, $$q(Y,Z1,Z2|X)$$, as the approximate posterior and minimize its ditance with the true posterior $$p(Y,Z1,Z2|X)$$. The simplest distance of choice is KL divergence i.e. $$D_KL = E_q [\log \frac{q(Y,Z1,Z2|X)}{P(Y,Z1,Z2|X)}]$$. 
- The KL divergence is intractable so we minimize its proxy i.e. the ELBO. 

<img src="/images/VAE_intuitions/vae_semi_M1_M2.png" alt="semi-supervised model inference" width="350" height="350">

- ELBO is the evidence lower bound:
$$\log p(X) =  \int_{Z1,Z2,Y} p(X,Y,Z1,Z2) $$

- We introduce the variational posterior:
$$\log p(X) =  \int_{Z1,Z2,Y} p(X,Y,Z1,Z2) \frac{q(Y,Z1,Z2|X)}{q(Y,Z1,Z2|X)} = \log E_q [\frac{p(X,Y,Z1,Z2)}{q(Y,Z1,Z2|X)}]$$

- Using the concavity of the $$\log$$ function and Jensen's inequality $$\log$$ goes into the expectation to provide a lower bound:
$$\log p(X) >= E_q[\log \frac{p(X,Y,Z1,Z2)}{q(Y,Z1,Z2|X)}]; \n \log p(X) >= E_q[\log p(X,Y,Z1,Z2)] - E_q [q(Y,Z1,Z2|X)] $$

- Now that the general form of the ELBO is derived, using the assumed factorization of the joint distribution, $$p(X,Y,Z1,Z2) = p(Z2)p(Y)p(Z1|Z2,Y)p(X|Z1)$$, and the factorization assumptions for the approximate posterior, the ELBO can be further simplified.

- Looking at the inference model of the M1+M2 model above shows that we are assuming a mean field assumption on the variational posterior. The assumed factorization of the approximate posterior is $$q(Y,Z1,Z2|X) = q(X|Z1)q(Y|Z1)q(Z1|Y,Z2)$$. The beauty of the variational approach is that we do not need to specify a specific parametric form for $$q$$. We specify how it should factorize, but then the optimization problem determines the optimal probability distribution within those factorization constraints.

- If we plug the factorizations of the generative and the inference models into the ELBO, we can derive a more simplified version.


## Semi-Supervised learning with GANs
- [Salimans 2016](https://arxiv.org/abs/1606.03498) and [Odena 2016](https://arxiv.org/abs/1606.01583) Usually for discriminator we only have two labels (i.e. real/fake).  and independantly replaced the output of the discriminator with several different categories (i.e. real cat/real dog/fake)
    + With labeled data you train the discriminator with the right label
    + With unlabeled data, you say the sum of probabilities of all real classes should be high
    + With fake image, you say the probability of the fake class should be high

- [Good Semi-supervised Learning that Requires a Bad GAN](https://arxiv.org/abs/1705.09783): Semi-supervised learning methods based on GANs work well, but it is not clear 1) how the discriminator benefits from joint training with a generator, and 2) why good semi-supervised classification performance and a good generator cannot be obtained at the same time. Theoretically, we show that given the discriminator objective, good semisupervised learning indeed requires a bad generator, and propose the definition of a preferred generator.


- [PixelGAN autoencoder](https://arxiv.org/abs/1706.00531) decoder is a PixelCNN conditioned on a latent code, and the encoder is an adversarial autoencoder with GAN loss to impose a prior on the latent code. semi-supervised learning can be done using a PixelGAN with a categorical prior. First the PixelGAN is trained on unlabeled data then the decoder is refined using labeled data.



