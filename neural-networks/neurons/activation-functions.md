---
title: activation functions
---

![](./activation_functions.png "Image showing common activation functions")
Activation functions are called on the dot product of the weights and their inputs plus the bias of a neuron. This function helps us control the types of outputs we want to come from our neuron.

When using activation functions its important to look at how the weights of that neuron affect the activation function. For example if you're using $$tanh$$ which squashes your values to $$[-1,1]$$ and you have a lot of values $$>10$$ or $$<10$$ you'll have a lot of values that are $$-1$$ and $$1$$ which basically makes the neuron useless. You can [visualize this in PyTorch](../pytorch#visualize-activations) quite nicely as well. Things like [normalization](./normalization.md) and standardization come in handy here.

Some further reading is available [here](https://arxiv.org/abs/1502.01852).

The reason we need activation functions (which are also called non-linearities) is because if we only have linear layers, no matter how many we add, the all collapse into a single linear equation. Which means the network is basically useless. We add non-linearities to the network so that each layer adds a form of intelligence when combined with the other layers.

## Softmax

A [softmax activation function](https://en.wikipedia.org/wiki/Softmax_function) is a mathematical expression that will take the outputs from a layer or a neural network and distributed them into probabilities, such that all the outputs sum to equal 1. This is an easy way to add probability distribution to the output of your net.
