---
title: neurons
description: how neurons work in neural networks
---

Neurons are exactly what they sound like, the things in our brain!
![](./neuron_model.jpeg "Diagram of a neuron")

In machine learning, we model these in neural networks to simulate how the brain works. Neurons take in a series of values (x) and weights (w), which are individually multiplied and then added. The neuron fires by taking these and adding the bias of the neuron (how trigger happy it is) and passing it through an [activation function](./activation-functions) which helps squash the values to something like -1 to 1. This is usually `tanh` or a `sigmoid` function.

## Weights

The weights for each input of a neuron are arbitrarily chosen. There's probably a whole field of mathematics that goes into determining the best starting weights, but at this point for me, it's random. Then through the process of training, these weights get adjusted to try and fit our loss function.

### Logits

Usually when starting you'll choose random numbers as weights, these numbers can be either positive or negative. Because of this, we'll commonly get the exponent of the values by doing $$e^x$$. Then all negative numbers will be $$[0,1]$$ and all positive numbers will be $$(1,inf]$$. These are commonly referred to as [Logits](https://www.linkedin.com/posts/mwitiderrick_what-are-logits-in-deep-learning-logits-activity-7084819307959902209-UUGe/).

> Logits are the outputs of a neural network before the [activation function](./activation-functions) is applied. They are the unnormalized probabilities of the item belonging to a certain class.

In the case of our [bi-gram NN model](../types/multi-layer-perceptron#single-layer-perceptron) the logits represent for each input character, the probabilities of each output character in the set.

## Biases

Much like [weights](#weights), biases are also randomly chosen and updated throughout the training process to try and adjust the activation of that neuron to fit our loss function.

### Random math

It can be useful to plot the values of your activation functions and gradients. You ideally want to see a gaussian distribution when doing so because that means your network doesn't have a ton of dead neurons.

## Layers

Idk where to put this, but an interesting pattern was covered, which is during training each layer is commonly composed of 3 parts. A convolutional/weight part, a normalization part, and a non-linearity part.
