---
title: types
description: different types of neural networks
---

# types of neural networks

Introductory explanations of the different types of model types that exist. Notes that apply to multiple types will commonly end up in this document. Models isn't necessarily the best word for everything in here, but I'll take it.

## Start and stop tokens

You'll typically create a special start and end character. Examples being something like `<S>` and `<S>` and this helps your model know when you're beginning a sequence or ending a sequence. However, you always know when you have or have not started processing a word, which means you don't need unique characters for start and stop, you can use the same character. Based on where you are in your processing you can contextually know whether it signifies starting or stopping.

## Model smoothing

This is the process of replacing zeros in your model with ones so that you don't ever run into cases where there is 0% chance of something resulting in a loss of infinity. There are likely more uses of this term, but so far this is the only one I've encountered.
