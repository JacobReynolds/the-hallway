---
title: attention
description: listen up, class
---

Attention, or self-attention, is the mechanism that makes current transformer models so powerful. It gives a neural network the ability to take into account previous values of the context to determine the current values.

## Elementary, my dear Watson

A simple example is looking at your current batch and average the embedding values for all tokens that have preceded you.

==- Code example

```python
import torch
torch.manual_seed(1)

B, T, C = 4, 8, 2

x = torch.randn(B, T, C)
xbowb = torch.zeros((B, T, C))
for batch in range(B):
    for time in range(T):
        xprev = x[batch][:time+1] # (T, C)
        xbowb[batch, time] = torch.mean(xprev, dim=0)
```

===
We can go ahead and remove the batch dimensions right now, since they only add complexity

==- Code example

```python
T, C = 8, 2
torch.manual_seed(1)
xbow = torch.randn((T, C))
for time in range(T):
    xprev = xbow[:time+1] # (T, C)
    x = torch.mean(xprev, dim=0)
    xbow[time] = x
```

===
And because math is math, we can actually express this as a simple matrix multiplication using `torch.tril`. Using `tril` and then averaging across it makes sure that when we perform the dot product, we're only averaging the previous values up to that row.

==- Code example

```python

a = torch.tril(torch.ones(T, T))
print(a)
# tensor([[1., 0., 0., 0., 0., 0., 0., 0.],
#         [1., 1., 0., 0., 0., 0., 0., 0.],
#         [1., 1., 1., 0., 0., 0., 0., 0.],
#         [1., 1., 1., 1., 0., 0., 0., 0.],
#         [1., 1., 1., 1., 1., 0., 0., 0.],
#         [1., 1., 1., 1., 1., 1., 0., 0.],
#         [1., 1., 1., 1., 1., 1., 1., 0.],
#         [1., 1., 1., 1., 1., 1., 1., 1.]])

a = a / a.sum(1, keepdim=True)
print(a)
# tensor([[1.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000],
#         [0.5000, 0.5000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000],
#         [0.3333, 0.3333, 0.3333, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000],
#         [0.2500, 0.2500, 0.2500, 0.2500, 0.0000, 0.0000, 0.0000, 0.0000],
#         [0.2000, 0.2000, 0.2000, 0.2000, 0.2000, 0.0000, 0.0000, 0.0000],
#         [0.1667, 0.1667, 0.1667, 0.1667, 0.1667, 0.1667, 0.0000, 0.0000],
#         [0.1429, 0.1429, 0.1429, 0.1429, 0.1429, 0.1429, 0.1429, 0.0000],
#         [0.1250, 0.1250, 0.1250, 0.1250, 0.1250, 0.1250, 0.1250, 0.1250]])

torch.manual_seed(1)
b = torch.randn((T, C))
c = a @ b
print(c)
# tensor([[-1.5256, -0.7502],
#         [-1.0898, -1.1799],
#         [-0.7599, -0.9896],
#         [-0.8149, -1.1445],
#         [-0.7943, -0.8549],
#         [-0.7915, -0.7543],
#         [-0.7102, -0.4055],
#         [-0.5929, -0.2964]])

print(torch.allclose(xbow, c))
# True
```

===
