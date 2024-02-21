---
title: neuron
description: how neurons work in neural networks
---

# Neuron

Neurons are exactly what they sound like, the things in our brain!
![](./neuron_model.jpeg "Diagram of a neuron")

In machine learning, we model these in neural networks to simulate how the brain works. Neurons take in a series of values (x) and weights (w), which are individually multiplied and then added. The neuron fires by taking these and adding the bias of the neuron (how trigger happy it is) and passing it through an [activation function](#activation-functions) which helps squash the values to something like -1 to 1. This is usually `tanh` or a `sigmoid` function.

## Weights

The weights for each input of a neuron are arbitrarily chosen. There's probably a whole field of mathematics that goes into determining the best starting weights, but at this point for me, it's random. Then through the process of training, these weights get adjusted to try and fit our loss function.

### Logits

Usually when starting you'll choose random numbers as weights, these numbers can be either positive or negative. Because of this, we'll commonly get the exponent of the values by doing $$e^x$$. Then all negative numbers will be $$[0,1]$$ and all positive numbers will be $$(1,inf]$$. These are commonly referred to as [Logits](https://www.linkedin.com/posts/mwitiderrick_what-are-logits-in-deep-learning-logits-activity-7084819307959902209-UUGe/).

> Logits are the outputs of a neural network before the [activation function](#activation-functions) is applied. They are the unnormalized probabilities of the item belonging to a certain class.

In the case of our [bi-gram NN model](../model-types/character-level#bi-gram-converted-to-a-single-layer-neural-network) the logits represent for each input character, the probabilities of each output character in the set.

## Biases

Much like [weights](#weights), biases are also randomly chosen and updated throughout the training process to try and adjust the activation of that neuron to fit our loss function.

## Activation functions

![](./activation_functions.png "Image showing common activation functions")
Activation functions are called on the dot product of the weights and their inputs plus the bias of a neuron. This function helps us control the types of outputs we want to come from our neuron.

When using activation functions its important to look at how the weights of that neuron affect the activation function. For example if you're using $$tanh$$ which squashes your values to $$[-1,1]$$ and you have a lot of values $$>10$$ or $$<10$$ you'll have a lot of values that are $$-1$$ and $$1$$ which basically makes the neuron useless. You can [visualize this in PyTorch](../../frameworks/pytorch#visualize-activations) quite nicely as well. Things like [normalization](#normalization) and standardization come in handy here.

Some further reading is available [here](https://arxiv.org/abs/1502.01852).

The reason we need activation functions (which are also called non-linearities) is because if we only have linear layers, no matter how many we add, the all collapse into a single linear equation. Which means the network is basically useless. We add non-linearities to the network so that each layer adds a form of intelligence when combined with the other layers.

### Normalization

It's important to look at how you initialize the values for your neurons. You essentially want them to be gaussian at the beginning so everything has a fair chance of training. If you look at the following, already after one layer, our standard deviation has almost tripled after performing the dot product.

```python
g = torch.Generator().manual_seed(1)
x = torch.randn(1000, 10, generator=g)
w = torch.randn(10, 200, generator=g)
print(w)
y = x @ w
print(x.mean(), x.std())
# tensor(-0.0038) tensor(1.0057)

print(w.mean(), w.std())
# tensor(0.0003) tensor(1.0115)

print(y.mean(), y.std())
# tensor(-0.0064) tensor(3.2211)
```

To fix this, the referenced paper above ended up in the addition of the [kaiming_normal operation](https://pytorch.org/docs/stable/nn.init.html), which divides the weights by the square root of the number of inputs to each neuron. It's a bit different depending on exactly which non-linearity you're using. For simplicity though you can commonly divide your weights by the square root of the fan-in, or the number of inputs to your neuron.

```python
g = torch.Generator().manual_seed(1)
x = torch.randn(1000, 10, generator=g)
w = torch.randn(10, 200, generator=g) / 10**.5
print(w)
y = x @ w
print(x.mean(), x.std())
# tensor(-0.0038) tensor(1.0057)

print(w.mean(), w.std())
# tensor(9.0598e-05) tensor(0.3199)

print(y.mean(), y.std())
# tensor(-0.0020) tensor(1.0186)
```

Multiplying your weights by this changes the standard deviation because if you look at a layer with standard deviation 1 and multiply it by `.2` the standard deviation is now `.2`.

```python
g = torch.Generator().manual_seed(1)
w = torch.randn(10, 200, generator=g)
print(w.std())
# tensor(1.0293)

w = w * .2
print(w.std())
# tensor(0.2059)
```

So following that, we can set the standard deviation of a gaussian distribution by multiplying it by whatever our ideal distribution is. The kaiming documentation gives us the equation

$$
std = \frac{gain}{\sqrt{fan\_mode}}
$$

where gain is precalculated by them in the docs. For $$tanh$$ it is $$\frac{5}{3}$$. And `fan_mode` is the number of axons of the neuron. Applying this to our function

```python
g = torch.Generator().manual_seed(1)
w = torch.randn(10, 200, generator=g) * ((5/3)/10**.5)

print(w.std())
# tensor(.3084)
```

So if doing it manually we would want to multiply our initial weights of a neuron with 10 axons by `.3084` to have a more normal standard deviation after passing through the activation function. This helps with tanh because it's a squashing function, so we want to amplify the values a bit, to fight back against it squashing everything inward. This normalization is typically fixed by applying something like [batch normalization](#batch-normalization) to each layer, reducing the important of the initial weights values.

### Batch normalization

A concept published in [this paper](https://arxiv.org/abs/1502.03167), provides a mechanism so that you don't have to normalize your weights at initialization but instead normalize them through the training cycle.

It's pretty complex and I don't fully understand it, but one critique of it is since it's working with the mean and standard deviation of each layer, it's coupling all of the inputs during training. So then independent inputs now have a dependency on each other and can affect training. What does that mean? I have no clue.

Another weird trait is that due to the way you calculate batch normalization, the biases actually ended up being effectively zeroed out. So when doing batch normalization you don't use bias in your neurons. It's important to remember the bias values wouldn't actually be zero, but because you're adding a constant value, the subtraction of the mean in the normalization removes any effect that bias would have.

You apply batch normalization on any layers of your choosing before or after the activation functions have been calculated, it's most common to do before though. You generate the normalization with the following

$$
z = \frac{x - m}{s}\newline
output = (z * g) + b\newline
x = \text{data point}\newline
m = \text{mean of the data set}\newline
s = \text{standard deviation}\newline
g = \text{arbitrary parameter}\newline
b = \text{arbitrary parameter}
$$

### Inputs

It's also common to preform normalization and standardization of your inputs, so that they follow a more equal distribution as well. A good analogy is miles driven, if some users drove 100 miles and others drove 100,000 that creates a significantly large data set that could have large imapcts on the gradients of the neural network. We can normalize them by squashing them down to 0-1 and standardize them by the equation in [Batch Normalization](#batch-normalization)

This is discussed in [this video](https://www.youtube.com/watch?v=dXB-KQYkzNU). Normalization and standardization seem to be used somewhat interchangeably.

### Softmax

A [softmax activation function](https://en.wikipedia.org/wiki/Softmax_function) is a mathematical expression that will take the outputs from a layer or a neural network and distributed them into probabilities, such that all the outputs sum to equal 1. This is an easy way to add probability distribution to the output of your net.

### Random math

It can be useful to plot the values of your activation functions and gradients. You ideally want to see a gaussian distribution when doing so because that means your network doesn't have a ton of dead neurons.

## Layers

Idk where to put this, but an interesting pattern was covered, which is during training each layer is commonly composed of 3 parts. A convolutional/weight part, a normalization part, and a non-linearity part.
