---
title: Gradient Descent
parent: Foundation
grand_parent: beep boop
layout: default
math: katex
---

# Gradient Descent

Gradient descent is the process fine tuning the weights and biases of a neural network to minimize our [loss function](../loss/).

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

If we increase $$p.data$$, that will lower the loss function. And just like the previous example $$-0.1 * -0.41 = 0.041$$ which will end up increasing $$p.data$$ and lowering the resulting loss function. The sign of $$p.data$$ actually has no effect here, it's only the sign of $$p.grad$$ that matters. And we manage that by basically inverting it by multiplying with $$-0.1$$. If we were instead looking to maximize the loss function, we'd multiple by $$+0.1$$
