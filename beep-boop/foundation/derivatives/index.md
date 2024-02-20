---
title: Derivatives
parent: Foundation
grand_parent: beep boop
layout: default
description: take me back to calc 1
math: katex
---

# Derivatives

This all gets a bit fuzzy for me. I never took multi-variable calculus and neural networks seem to rely on them for partial derivatives..._I think_. But the concepts outlined below should be a good summary. I still have to check [these](https://www.youtube.com/watch?v=uXt8qF2Zzfo) [out](https://www.jeremyjordan.me/neural-networks-training/) to stress test my knowledge here.

## Basics

The derivative of the function $$f(x)$$ is the slope at point $$x$$, or how much the value changes at that point when increasing x by tiny amounts. In more explicit terms, it’s the value of:

$$
f'(x) = \lim{h \to 0} \frac{f(x+h) - f(x)}{h}
$$

For the function $$3x^2 - 4x + 5$$ we can see that the following is true

```python
def f(x):
	return 3*x**2 - 4*x + 5

x = 3
h = 0.001
f(x)
# 20.0
f(x + h)
# 20.014003000000002
h = 0.0000001
# 20.00000140000003
```

So it’s increasing ever so slightly in the positive direction. To get the actual slope, we need to get the [rise over run](https://www.khanacademy.org/math/algebra/x2f8bb11595b61c86:linear-equations-graphs/x2f8bb11595b61c86:slope/a/slope-review).

```python
(f(x+h) - f(x)) / h
# 14.000000305713911
```

Which if we take the derivative of $$f(x)$$ where $$x = 3$$ we get $$f'(x) = 6x - 4 = 6*3 - 4 = 14$$ which matches our equations.

This is important, because as we start making complex chains of nodes (a neural net), we want to know the derivative of the output wrt the individual nodes. That way we can tell how we have to change that node to manipulate the output of the function.

![Screenshot showing gradient nodes](./derivatives_nodes.png)

This is a good example, where we would want to know the derivative of L wrt c, so we know how to change c to impact L.

### With respect to

I struggled a bit to understand/remember what derivatives of a variable w.r.t. a function/another variable _means_.

$$
d = b \cdot a + c
$$

$$
\frac{dd}{da} = b
$$

You would say this is “The derivative of $$d$$ (where d is the function) w.r.t. $$a$$. And what that means is whenever I change $$a$$, how does that change the output of $$d$$? Using the [Constant Multiple Rule](https://www.khanacademy.org/math/old-ap-calculus-ab/ab-derivative-rules/ab-basic-diff-rules/a/basic-differentiation-review) we know that the derivative of a variable times a function is that variable times the derivative of the function.

$$
\frac{d}{dx}(k*f(x)) = k * \frac{d}{dx}(f(x))
$$

Why do we know that? Because some mathematician made it up. So in the above, $$k=b$$ and $$f(x) = a$$ which gives us $$f'(x) = a' = 1$$ which leaves us with $$b$$. So each time we increase $$a$$ by 1, $$d$$ increases by $$b$$. What happened to $$c$$? Since it's not one of the variables being operated on, it gets treated as a constant and the derivative of a constant is 0. That's a property of taking a [partial derivative](https://www.khanacademy.org/math/multivariable-calculus/multivariable-derivatives/partial-derivative-and-gradient-articles/a/introduction-to-partial-derivatives).

Something important to think about is that in $$d$$ if we changed $$c$$ to $$c^2$$ the outcome is still the same. Because w.r.t $$a$$, $$c$$ is constant. Meaning changing $$a$$ has no impact on $$c$$. However if the equation was something like

$$
d = a \cdot b + a \cdot c^2
$$

We would then end up with a derivative of

$$
\frac{dd}{da} = b + 2c
$$

## Chain Rule

The chain rule, in simple terms, says that if you have a composite function $$f(g(x))$$ and want to take the derivative of it, you can do $$f'(g(x)) * g'(x)$$ . I.e. the derivative of the composite function is the inner function within the derivative of the outer function, multiplied by the derivative of the inner function.

### As it relates to [back propagation](../back-propagation/)

More simply, as we navigate through back propagation, we will want to identify how one node affects the loss function. But this node could be one of many.

![Screenshot showing nodes](./chain_rule_grad.png)

Using the above example, the local derivative of $$c \space w.r.t \space d$$ is 1, because it’s [addition](#addition). But how does $$c$$ affect $$L$$? The [Chain Rule](https://en.wikipedia.org/wiki/Chain_rule) gives us this. And what it says is, using the above variable names

$$
\frac{dL}{dc} = \frac{dL}{dd} \cdot \frac{dd}{dc}
$$

Which in this case is $$-2 \cdot 1 = -2$$. Why is $$\frac{dd}{dc}=1$$? See [addition](#addition). It's even nicer because as we get further along the back propagation we only need to multiply the local derivative by the gradient of the grandparent node. Take the following:
![Larger screenshot showing nodes](./chain_rule_ext.png)
If we want to do $$\frac{dL}{da}$$ we would only need to do $$\frac{de}{da} \cdot \frac{dd}{de}$$, since it's all multiplication we don't have to chain it all the way back to L.

Another cool thing here is that since the back propagation only relies on multiplying the local derivative with the grandparent derivative, the calculation can be arbitrarily complex. It doesn't have to be $$+$$ or $$*$$ between nodes, it can be anything as long as we can derive it. This is helpful when doing back propagation on a [neuron](../neuron) that has an activation function, so we can derive the value inclusive of the activation function.

### Addition

Doing the derivative of an additive operation w.r.t. a node is pretty easy. It equals 1. Why is that true? Take these nodes for example
![Screenshot showing nodes](./chain_rule_grad.png)

$$
d = c + e
$$

$$
\frac{dd}{dc} = \frac{d}{dd} c + \frac{d}{dd} e
$$

$$
\frac{dd}{dc} = \frac{d}{dd} c + 0
$$

$$
\frac{dd}{dc} = 1 + 0 = 1
$$

Why did $$e$$ become $$0$$? Remember that's a property of taking a [partial derivative](https://www.khanacademy.org/math/multivariable-calculus/multivariable-derivatives/partial-derivative-and-gradient-articles/a/introduction-to-partial-derivatives) wherein any variables we are not currently working with become constants. And the derivative of a constant is 0.

Also don't forget that $$1$$ is only the local derivative, we have to apply the [chain rule](#chain-rule) to get the gradient. Doing that we get $$1 \cdot -2 = -2$$.

## References

[https://www.khanacademy.org/math/ap-calculus-ab/ab-differentiation-2-new/ab-3-1a/a/chain-rule-review](https://www.khanacademy.org/math/ap-calculus-ab/ab-differentiation-2-new/ab-3-1a/a/chain-rule-review)
