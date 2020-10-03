---
layout: post
title:  "Derivation Proofs"
date:   2020-10-02 19:09:05 +0200
categories: Random-Facts math calculation derivation
usemathjax: true
---
## Basic
The Derivation of a function can be expressed as follows.

$$f'(x) = \lim_{\Delta x\rightarrow x_0}\frac{f(x+\Delta x)-f(x)}{\Delta x}$$

The formular for an power function is as follows.

$$f(x)=x^n$$

Inserting the power function creates the following expression.

$$f'(x) = \lim_{\Delta x\rightarrow 0} \frac{(x+\Delta x)^n-x^n}{\Delta x}$$ 

The binominal formular can be expanded.

$$f'(x) = \lim_{\Delta x\rightarrow 0} \frac{\frac{n!}{0!(n-0)!}x^n+\frac{n!}{1!(n-1)!}x^{n-1}\Delta x+\frac{n!}{2!(n-2)!}x^{n-2}\Delta x^2+...+\frac{n!}{n!(n-n)!}x^{n-n}\Delta x^n - x^n}{\Delta x}$$ 

The first term result in $$\frac{n!}{0!(n-0)!}x^n = x^n$$ and so it gets removed and so it becomes possible to perform the division with $$\Delta x$$. Then inserting the limit $$\Delta x \rightarrow 0$$ results in the following equation.

$$f'(x) = \frac{n!}{1!(n-1)!}x^{n-1} = nx^{n-1}$$

## Exponential
The formular for an exponential function is as follows.

$$f'(x) = a^x$$

Rewriting it enables a simple derivation 

$$f(x) = a^x = e^{\ln(a)x}$$

Derivation is as follows

$$f'(x) = \frac{d}{dx}a^x=\frac{d}{dx}e^{\ln(a)x}=\ln(a)e^{\ln(a)x}=\ln(a)a^x=\ln(a)a^x$$

## Logarithmic
The formular for an logarithmic function is as follows.

$$f(x) = \log_a x$$

$$\frac{d}{dx}a^{f(x)} = \frac{d}{dx}x$$

$$a^{f(x)}\ln a f'(x) = 1$$

Inserting the function leads to as follows

$$a^{\log_a x}\ln a f'(x) = 1 = x\ln a f'(x)$$

$$f'(x) = \frac{1}{x\ln a}$$

Inserting $$a = e$$ results in the following for $$f(x) = \ln x$$

$$f'(x) = \frac{1}{x}$$

## Trigonometric
### Sine
Inserting sine into the basic formular results to as follows

$$f(x) = \sin x$$

$$f'(x) = \lim_{\Delta x\rightarrow 0}\frac{\sin(x+\Delta x)-\sin(x)}{\Delta x}$$

Using the [Sum-to-product](https://en.wikipedia.org/wiki/List_of_trigonometric_identities#Product-to-sum_and_sum-to-product_identities) identity we get the following

$$f'(x) = \lim_{\Delta x\rightarrow 0}\frac{2\sin\left(\frac{x + \Delta x - x}{2}\right)\cos\left(\frac{x + \Delta x + x}{2}\right)}{\Delta x} = \lim_{\Delta x\rightarrow 0}\frac{\sin\left(\frac{\Delta x}{2}\right)}{\frac{\Delta x}{2}} * \lim_{\Delta x\rightarrow 0}\cos\left(x + \frac{\Delta x}{2}\right)$$

Using the [Squeeze theorem](https://en.wikipedia.org/wiki/Squeeze_theorem#Second_example) we can get rid of sine and insert $$\Delta x \rightarrow 0$$.

$$f'(x) = \cos(x)$$

### Cosine
Cosine can be expressed as follows

$$f(x) = \cos x = \sin\left(\frac{\pi}{2} - x\right)$$

Deriving it creates the following

$$f'(x) = \frac{d}{dx}\sin\left(\frac{\pi}{2} - x\right) = \cos\left(\frac{\pi}{2} - x\right) * \frac{d}{dx}\left[\frac{\pi}{2} - x\right]$$

$$f'(x) = \sin x * (-1) = -\sin x$$

### Inverse
The inverse of sine is expressed as follows

$$f(x) = \arcsin x$$

If $$f(x) = \arcsin x$$ then the following happens

$$\sin f(x) = x$$

$$\frac{d}{dx}\sin f(x) = \frac{d}{dx}x = \cos f(x)f'(x) = 1$$

Inserting the original equation lead to the following

$$1 = \cos\arcsin xf'(x)$$

$$f'(x) = \frac{1}{\cos\arcsin x}$$

Using a [Pythagorean identity](https://en.wikipedia.org/wiki/List_of_trigonometric_identities#Pythagorean_identities) we can substitute for cosine

$$f'(x) = \frac{1}{\sqrt{1-\sin^2\arcsin x}} = \frac{1}{\sqrt{1-x^2}}$$