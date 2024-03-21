---
title: attention
description: listen up, class
---

Attention, or self-attention, is the mechanism that makes current transformer models so powerful. It gives a neural network the ability to take into account previous values of the context to determine the current values.

## Simple averaging

### Batched average

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

### Using `torch.tril`

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

### Using `softmax`

One more alternative is to instead use `softmax`

==- Code example

```python
T, C = 8, 2
torch.manual_seed(1)

x = torch.randn(T, C)
tril = torch.tril(torch.ones(T, T))

wei = torch.zeros((T,T))
wei = wei.masked_fill(tril == 0, -torch.inf)
print(wei)
# tensor([[0., -inf, -inf, -inf, -inf, -inf, -inf, -inf],
#         [0., 0., -inf, -inf, -inf, -inf, -inf, -inf],
#         [0., 0., 0., -inf, -inf, -inf, -inf, -inf],
#         [0., 0., 0., 0., -inf, -inf, -inf, -inf],
#         [0., 0., 0., 0., 0., -inf, -inf, -inf],
#         [0., 0., 0., 0., 0., 0., -inf, -inf],
#         [0., 0., 0., 0., 0., 0., 0., -inf],
#         [0., 0., 0., 0., 0., 0., 0., 0.]])

wei = torch.softmax(wei, dim=1)
print(wei)
# tensor([[1.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000],
#         [0.5000, 0.5000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000],
#         [0.3333, 0.3333, 0.3333, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000],
#         [0.2500, 0.2500, 0.2500, 0.2500, 0.0000, 0.0000, 0.0000, 0.0000],
#         [0.2000, 0.2000, 0.2000, 0.2000, 0.2000, 0.0000, 0.0000, 0.0000],
#         [0.1667, 0.1667, 0.1667, 0.1667, 0.1667, 0.1667, 0.0000, 0.0000],
#         [0.1429, 0.1429, 0.1429, 0.1429, 0.1429, 0.1429, 0.1429, 0.0000],
#         [0.1250, 0.1250, 0.1250, 0.1250, 0.1250, 0.1250, 0.1250, 0.1250]])

xbow = wei @ x
print(xbow)
# tensor([[-1.5256, -0.7502],
#         [-1.0898, -1.1799],
#         [-0.7599, -0.9896],
#         [-0.8149, -1.1445],
#         [-0.7943, -0.8549],
#         [-0.7915, -0.7543],
#         [-0.7102, -0.4055],
#         [-0.5929, -0.2964]])
```

===

## Self-attention

I really don't understand this yet, but here's some of the code and specific notes from Andrej's video.

==- Code example

```python
B, T, C = 4, 8, 32
torch.manual_seed(1)
x = torch.randn(B, T, C)

head_size = 16
key = torch.nn.Linear(C, head_size, bias=False)
query = torch.nn.Linear(C, head_size, bias=False)
value = torch.nn.Linear(C, head_size, bias=False)
k = key(x)   # (B, T, 16)
q = query(x) # (B, T, 16)
wei = q @ k.transpose(-2, -1) # (B, T, 16) @ (B, 16, T) ---> (B, T, T)

tril = torch.tril(torch.ones((T, T)))
wei = wei.masked_fill(tril == 0, float(-torch.inf))
wei = torch.nn.functional.softmax(wei, dim=-1)
v = value(x)
out = wei @ v
```

===

- Attention is a communication mechanism. Can be seen as nodes in a directed graph looking at each other and aggregating information with a weighted sum from all nodes that point to them, with data-dependent weights.
- There is no notion of space. Attention simply acts over a set of vectors. This is why we need to positionally encode tokens.
- Each example across batch dimension is of course processed completely independently and never "talk" to each other
- In an "encoder" attention block just delete the single line that does masking with `tril`, allowing all tokens to communicate. This block here is called a "decoder" attention block because it has triangular masking, and is usually used in autoregressive settings, like language modeling.
- "self-attention" just means that the keys and values are produced from the same source as queries. In - "cross-attention", the queries still get produced from x, but the keys and values come from some other, external source (e.g. an encoder module)
- "Scaled" attention additionally divides `wei` by 1/sqrt(head_size). This makes it so when input Q,K are unit variance, wei will be unit variance too and Softmax will stay diffuse and not saturate too much. Illustration below

The final note can be done by doing

```python
wei = q @ k.transpose(-2, -1) * headsize**-.5
```
