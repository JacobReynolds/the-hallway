---
title: Neuron
layout: default
parent: Foundation
description: how neurons work in neural networks
grand_parent: beep boop
---

# Neuron

Neurons are exactly what they sound like, the things in our brain!
![Diagram of a neuron](./neuron_model.jpeg)

In machine learning, we model these in neural networks to simulate how the brain works. Neurons take in a series of values (x) and weights (w), which are individually multiplied and then added. The neuron fires by taking these and adding the bias of the neuron (how trigger happy it is) and passing it through an activation function which helps squash the values to something like -1 to 1. This is usually `tanh` or a `sigmoid` function.

## Weights

The weights for each input of a neuron are arbitrarily chosen. There's probably a whole field of mathematics that goes into determining the best starting weights, but at this point for me, it's random. Then through the process of training, these weights get adjusted to try and fit our loss function.

## Biases

Much like [weights](#weights), biases are also randomly chosen and updated throughout the training process to try and adjust the activation of that neuron to fit our loss function.
