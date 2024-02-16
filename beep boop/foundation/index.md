---
title: Foundation
nav_order: 1
layout: default
parent: beep boop
has_children: true
---

[Back propagation](https://neptune.ai/blog/backpropagation-algorithm-in-neural-networks-guide) uses the [Chain Rule](./chain-rule/) from calculus.

Neural networks **are just math**.

Tensors are basically scalar values from [micrograd](https://github.com/karpathy/micrograd), but put into arrays to take advantage of parallelism in computing.

Back propagation relies heavily on calculating [Derivatives](./derivatives/) to get the slope of a function, to understand how much different inputs affect the activation function. Each neuron has a gradient, and that gradient is the derivative of the loss function w.r.t. that node. I.e. what is the slope at that point.

Loss functions help us identify the gap between what we expect from a function and what we actually got from it. The lower the loss, the more accurate the function is
