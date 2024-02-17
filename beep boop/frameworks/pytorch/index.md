---
title: PyTorch
parent: Frameworks
layout: default
grand_parent: beep boop
---

# 🥧🔥 PyTorch

If you read the word pytorch over and over, it really starts to lose it's meaning. Freaks me out when words do that. [PyTorch](https://pytorch.org/) is a production-grade machine learning framework written in Python that aids you in building and training neural networks.

Here's a simple tree showing the equation and back propagation we're doing, alongside the torch code to calculate the same.

![DAG of nodes for a math equation](./equation.png)

```python
import torch
x1 = torch.Tensor([2.0]).double()                ; x1.requires_grad = True
x2 = torch.Tensor([0.0]).double()                ; x2.requires_grad = True
w1 = torch.Tensor([-3.0]).double()               ; w1.requires_grad = True
w2 = torch.Tensor([1.0]).double()                ; w2.requires_grad = True
b = torch.Tensor([6.8813735870195432]).double()  ; b.requires_grad = True
n = x1*w1 + x2*w2 + b
o = torch.tanh(n)

print(o.data.item())
o.backward()

print('---')
print('x2', x2.grad.item())
print('w2', w2.grad.item())
print('x1', x1.grad.item())
print('w1', w1.grad.item())

# 0.7071066904050358
# ---
# x2 0.5000001283844369
# w2 0.0
# x1 -1.5000003851533106
# w1 1.0000002567688737
```

You have to tell torch `x1.requires_grad = True` because they're leaf nodes and traditionally you don't want to calculate gradients for leaf nodes. My unconfirmed assumption, is because you usually aren't trying to change the inputs of the NN, you're trying to change the weights and biases of the [neurons](../../foundation/neuron/) inside of it.

## Tensors

Torch and other frameworks use the concept of a Tensor. A Tensor is an n-dimensional array of scalar values. This is done to take advantage of computer parallelism to speed up calculations.
