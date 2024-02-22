---
title: tokenizing
icon: arrow-switch
description: converting words to numbers for fun and profit
---

Neural networks don't actually understand the things they work on. They don't know the letter "a" exists, or the word "dog". All they know is the numeric representations of those words, which are commonly a combination of tokenizing and [encoding](../encoding/).

Tokenizing is taking these words and splitting them up into a repeatable and referenceable language. The simplest form of this is converting a sentence or word into its constituent characters.

==- Code example

```python
input = "the quick brown fox jumped over the lazy dog."

chars = sorted(list(set(input)))
print(''.join(chars))
#  .abcdefghijklmnopqrtuvwxyz

vocab_size = len(chars)
print(vocab_size)
# 27

stoi = dict([c, i] for i,c in enumerate(chars))
print(stoi)
# {' ': 0, '.': 1, 'a': 2, 'b': 3, 'c': 4, 'd': 5, 'e': 6, 'f': 7, 'g': 8, 'h': 9, 'i': 10, 'j': 11, 'k': 12, 'l': 13, 'm': 14, 'n': 15, 'o': 16, 'p': 17, 'q': 18, 'r': 19, 't': 20, 'u': 21, 'v': 22, 'w': 23, 'x': 24, 'y': 25, 'z': 26}

itos = dict([i, c] for i,c in enumerate(chars))
print(itos)
# {0: ' ', 1: '.', 2: 'a', 3: 'b', 4: 'c', 5: 'd', 6: 'e', 7: 'f', 8: 'g', 9: 'h', 10: 'i', 11: 'j', 12: 'k', 13: 'l', 14: 'm', 15: 'n', 16: 'o', 17: 'p', 18: 'q', 19: 'r', 20: 't', 21: 'u', 22: 'v', 23: 'w', 24: 'x', 25: 'y', 26: 'z'}
```

===

## Options

There are may options for how to tokenize your inputs, I'm still learning exactly what they all are. An example being [SentencePiece](https://github.com/google/sentencepiece) or [tiktoken](https://github.com/openai/tiktoken).
