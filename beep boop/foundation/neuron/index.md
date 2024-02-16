---
title: Neuron
layout: default
parent: Foundation
grand_parent: beep boop
---

# Neuron

Neurons are exactly what they sound like, the things in our brain!
![Diagram of a neuron](./neuron_model.jpeg)

In machine learning, we model these in neural networks to simulate how the brain works. Neurons take in a series of values (x) and weights (w), which are individually multiplied and then added. The neuron fires by taking these and adding the bias of the neuron (how trigger happy it is) and passing it through an activation function which helps squash the values to something like -1 to 1. This is usually `tanh` or a `sigmoid` function.
