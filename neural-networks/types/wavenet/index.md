---
title: wavenet
description: wavenet
order: 998
---

Still need to expand my thoughts here.

==- Code example

```python
import torch
import matplotlib.pyplot as plt
%matplotlib inline

torch.manual_seed(101)

words = open('names.txt', 'r').read().splitlines()
[training_set, validation_set, test_set] = torch.utils.data.random_split(words, [.8 ,.1 ,.1])

TOKEN='.'
chars = list(set(''.join(words)+TOKEN))
chars.sort()
itos = dict([(i,s) for i, s in enumerate(chars)])
stoi = dict([s, i] for i, s in enumerate(chars))
vocab_size = len(chars)
block_size = 8

def build_dataset(words):
    xs = []
    ys = []
    for word in words:
        context = [0]*block_size
        for i, ch in enumerate(word+TOKEN):
            ix = stoi[ch]
            xs.append(context)
            ys.append(ix)
            context = context[1:]+[ix]
    return torch.tensor(xs), torch.tensor(ys)

Xtr, Ytr = build_dataset(training_set)
Xval, Yval = build_dataset(validation_set)
Xtest, Ytest = build_dataset(test_set)

n_embd = 24
n_hidden = 128

# Departs from pytorch implementation https://youtu.be/t3YJ5hKiMQ0?si=dIVp0YFYBFH0NpdS&t=2620
class BatchNorm1d(torch.nn.Module):
  def __init__(self, dim, eps=1e-5, momentum=0.1):
    super(BatchNorm1d, self).__init__()
    self.eps = eps
    self.momentum = momentum
    self.training = True
    # parameters (trained with backprop)
    self.gamma = torch.ones(dim)
    self.beta = torch.zeros(dim)
    # buffers (trained with a running 'momentum update')
    self.running_mean = torch.zeros(dim)
    self.running_var = torch.ones(dim)

  def __call__(self, x):
    # calculate the forward pass
    if self.training:
      if x.ndim == 2:
        dim = 0
      elif x.ndim == 3:
        dim = (0,1)
      xmean = x.mean(dim, keepdim=True) # batch mean
      xvar = x.var(dim, keepdim=True) # batch variance
    else:
      xmean = self.running_mean
      xvar = self.running_var
    xhat = (x - xmean) / torch.sqrt(xvar + self.eps) # normalize to unit variance
    self.out = self.gamma * xhat + self.beta
    # update the buffers
    if self.training:
      with torch.no_grad():
        self.running_mean = (1 - self.momentum) * self.running_mean + self.momentum * xmean
        self.running_var = (1 - self.momentum) * self.running_var + self.momentum * xvar
    return self.out

  def parameters(self):
    return [self.gamma, self.beta]


class FlattenConsecutive(torch.nn.Module):
    def __init__(self, n):
      super(FlattenConsecutive, self).__init__()
      self.n = n

    def __call__(self, x):
        num_batches, num_characters, num_embeddings = x.shape
        x = x.view(num_batches, num_characters//self.n, num_embeddings*self.n)
        if x.shape[1] == 1:
          x = x.squeeze(1)
        self.out = x
        return self.out

model = torch.nn.Sequential(
    torch.nn.Embedding(vocab_size, n_embd),
    FlattenConsecutive(2), torch.nn.Linear(n_embd * 2, n_hidden, bias=False), BatchNorm1d(n_hidden), torch.nn.Tanh(),
    FlattenConsecutive(2), torch.nn.Linear(n_hidden * 2, n_hidden, bias=False), BatchNorm1d(n_hidden), torch.nn.Tanh(),
    FlattenConsecutive(2), torch.nn.Linear(n_hidden * 2, n_hidden, bias=False), BatchNorm1d(n_hidden), torch.nn.Tanh(),
    torch.nn.Linear(n_hidden, vocab_size)
)

with torch.no_grad():
    model[-1].weight.data *= .1

print(sum(p.nelement() for p in model.parameters()))
# 75811

for p in model.parameters():
  p.requires_grad = True

max_steps = 25000
batch_size = 32
for i in range(max_steps):
    ix = torch.randint(0, Xtr.shape[0], (batch_size, ))
    Xb, Yb = Xtr[ix], Ytr[ix]
    # forward pass
    x = model(Xb)
    loss = torch.nn.functional.cross_entropy(x, Yb)

    for p in model.parameters():
        p.grad = None
    loss.backward()

    lr = 0.1 if i<(max_steps*.75) else .01
    for p in model.parameters():
        p.data += -lr * p.grad

    if i%(max_steps/10) == 0:
        print(f"{i:7d}/{max_steps:7d}: loss={loss.item(): .4f}")

#       0/  25000: loss= 3.2859
#    2500/  25000: loss= 2.5645
#    5000/  25000: loss= 2.4274
#    7500/  25000: loss= 2.3042
#   10000/  25000: loss= 2.1279
#   12500/  25000: loss= 2.0867
#   15000/  25000: loss= 1.9211
#   17500/  25000: loss= 2.0363
#   20000/  25000: loss= 2.0193
#   22500/  25000: loss= 1.9068

model.eval()

# evaluate the loss
@torch.no_grad() # this decorator disables gradient tracking inside pytorch
def split_loss(split):
  x,y = {
    'train': (Xtr, Ytr),
    'val': (Xval, Yval),
    'test': (Xtest, Ytest),
  }[split]
  logits = model(x)
  loss = torch.nn.functional.cross_entropy(logits, y)
  print(split, loss.item())

split_loss('train')
split_loss('val')
split_loss('test')

# train 1.9855728149414062
# val 2.0416152477264404
# test 2.034022331237793

num_examples = 10
for _ in range(num_examples):
    CONTEXT = [0] * block_size
    out = []
    while True:
        x = model(torch.tensor([CONTEXT]))
        probs = torch.softmax(x, 1)
        ix = torch.multinomial(probs, num_samples=1, replacement=True).item()
        if ix == 0:
            break
        CONTEXT = CONTEXT[1:] + [ix]
        out.append(ix)
    print(''.join([itos[c] for c in out]))

# daorbose
# elinnah
# xaras
# bryston
# ambiya
# raliya
# zahaa
# eyin
# laui
# riyvanna
```

===
