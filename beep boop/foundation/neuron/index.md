---
title: Neuron
layout: default
parent: Foundation
description: how neurons work in neural networks
grand_parent: beep boop
math: katex
---

# Neuron

Neurons are exactly what they sound like, the things in our brain!
![Diagram of a neuron](./neuron_model.jpeg)

In machine learning, we model these in neural networks to simulate how the brain works. Neurons take in a series of values (x) and weights (w), which are individually multiplied and then added. The neuron fires by taking these and adding the bias of the neuron (how trigger happy it is) and passing it through an activation function which helps squash the values to something like -1 to 1. This is usually `tanh` or a `sigmoid` function.

## Weights

The weights for each input of a neuron are arbitrarily chosen. There's probably a whole field of mathematics that goes into determining the best starting weights, but at this point for me, it's random. Then through the process of training, these weights get adjusted to try and fit our loss function.

### Logits

Usually when starting you'll choose random numbers as weights, these numbers can be either positive or negative. Because of this, we'll commonly get the exponent of the values by doing $$e^x$$. Then all negative numbers will be $$[0,1]$$ and all positive numbers will be $$(1,inf]$$. These are commonly referred to as [Logits](https://www.linkedin.com/posts/mwitiderrick_what-are-logits-in-deep-learning-logits-activity-7084819307959902209-UUGe/).

> Logits are the outputs of a neural network before the activation function is applied. They are the unnormalized probabilities of the item belonging to a certain class.

In the case of our [bi-gram NN model](../model-types/character-level#bi-gram-converted-to-a-single-layer-neural-network) the logits represent for each input character, the probabilities of each output character in the set.

## Biases

Much like [weights](#weights), biases are also randomly chosen and updated throughout the training process to try and adjust the activation of that neuron to fit our loss function.
