---
title: Gradient Descent
parent: Foundation
grand_parent: beep boop
description: fine tuning a neural network
layout: default
math: katex
---

# Gradient Descent

Gradient descent is the process of fine tuning the weights and biases of [neurons](../neuron) in a neural network to minimize our [loss function](../loss/).

## Example

We do this by performing [back propagation](../back-propagation/) across something like a [multi-layer perceptron](../multi-layer-perceptron/) to [calculate](../derivatives/) the gradients of each [neuron](../neuron/). We do this so when we do a forward-pass through the MLP, we can compare the expected outputs against the actual outputs using a [loss function](../loss/). Gradient descent is then the process of adjusting the weights and biases of each neuron, to get our loss function as low as possible. The gradient of each neuron helps us understand whether to change the weights/biases of that neuron in a positive or negative direction to achieve the output we want.

Building off the [multi-layer perceptron](../multi-layer-perceptron/) implementation, we can perform gradient descent with the following:

```python
n = MLP(3, [4, 4, 1])
xs = [
  [2.0, 3.0, -1.0],
  [3.0, -1.0, 0.5],
  [0.5, 1.0, 1.0],
  [1.0, 1.0, -1.0],
]
ys = [1.0, -1.0, -1.0, 1.0]

for k in range(20):

  # forward pass
  ypred = [n(x) for x in xs]
  loss = sum((yout - ygt)**2 for ygt, yout in zip(ys, ypred))

  # backward pass
  for p in n.parameters():
    p.grad = 0.0
  loss.backward()

  # update
  for p in n.parameters():
    p.data += -0.1 * p.grad

  print(k, loss.data)
```

The reasoning for `-0.1` here is actually super important. We have to remember the goal of this gradient descent is to lower the value of the loss function as much as possible. So when tuning our weights, we want to tune them such that they _decrease_ the loss function. Luckily we know that the gradient will tell us how much that value will change the output. Let's look at some examples:

$$
p.grad = 0.41\newline
p.data = 0.88
$$

In this case, if we want to decrease the loss function, we want to decrease $$p.data$$, because $$p.grad$$ tells us that for every $$n$$ we increase $$p.data$$ the loss function changes by $$n \cdot 0.41$$. So it makes sense to instead do $$-0.1 * p.grad$$ here.

But what if the signs are different?

$$
p.grad = -0.41\newline
p.data = 0.88
$$

In this case, increasing $$p.data$$ decreases the loss function. If we do $$-0.1 \cdot -0.41$$ we get $$0.041$$ which will increase $$p.data$$ and further decrease the loss function.

One more

$$
p.grad = -0.41\newline
p.data = -0.88
$$

If we increase $$p.data$$, that will lower the loss function. And just like the previous example $$-0.1 * -0.41 = 0.041$$ which will end up increasing $$p.data$$ and lowering the resulting loss function. The sign of $$p.data$$ actually has no effect here, it's only the sign of $$p.grad$$ that matters. And we manage that by basically inverting it by multiplying with $$-0.1$$. If we were instead looking to maximize the loss function, we'd multiple by $$+0.1$$.

I got confused here for a bit trying to understand how we *know* that decreasing $$p.data$$ would decrease the loss function. What if the output is too low, wouldn't we want to increase the data? It's important to remember the loss function is almost like a continuation of the neural network. You take the outputs from the network and calculate the loss functions with those. So the final item in the equation is actually the output of the loss function, not the output of the neural net. That means our gradients are now directly tied to the loss function, not the outputs of the NN, due to performing back propogation starting with the loss function. 

### Zero grad

You'll notice in the backwards pass above, we reset the gradient before each backwards pass. This is because after we change the weights and biases of a neuron, the gradient also changes and the previous values have no effect on it. So we reset all grads to zero so the next backwards pass can recalculate them from scratch.

## Choosing a learning rate

When figuring out the learning rate to multiply your gradient by, it's important not to have too large of a value as you could overstep the optimal point. But also too small of a value can cause learning to take forever.

As the number of iterations increases, it's common to use a strategy called "Learning Rate Decay" that lowers the learning rate as you get further into your training.

## At scale

When performing gradient descent at scale, it's common to only analyze a small subset of the total neurons to save on computation time. This means you'll choose a batch of neurons/layers, perform a forward pass on them, calculate the [loss function](../loss) on that batch, and update gradients within that batch accordingly.
