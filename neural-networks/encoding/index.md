---
title: encoding
description: handling features for a model
---

# Encoding

When feeding data into a neural network, we will need to represent out inputs (or features) in a more machine-friendly way. There are many ways to do this.

## One-hot encoding

This transforms our input set (e.g. all characters a-z) into an array that is 26-characters long. All values will be zero, except for the character we are feeding in, which will be a one. This is because the neural network doesn't give a shit about ascii or human-readable characters, it's purely operating on the underlying neurons and their inputs. See [PyTorch](../../frameworks/pytorch/) for more info.
