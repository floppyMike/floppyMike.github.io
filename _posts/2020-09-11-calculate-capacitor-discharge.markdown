---
layout: post
title:  "Calculate capacitor discharge voltage"
date:   2020-09-10 14:19:45 +0000
categories: capacitor math
usemathjax: true
---
In a series connection the relationship between the voltage of the capacitor and resistor can be described as follows.

$$U_{cap} + U_R = 0$$

Using [Ohm's Law](https://en.wikipedia.org/wiki/Ohm%27s_law) $$R = \frac{U}{I}$$ we can substitute for $$U_R$$.

$$U_{cap} + IR = 0$$

$$U_{cap} = -IR$$

We can substitute the electric current with its definition $$I = \frac{dQ}{dt}$$.

$$U_{cap} = -\frac{dQ}{dt} * R$$

The voltage of the capacitor has the following [relationship](https://en.wikipedia.org/wiki/Capacitance#Self_capacitance) $$U = \frac{Q}{C}$$.

$$\frac{Q}{C} = -\frac{dQ}{dt} * R$$

Now we can define a integral with $$Q_0, Q_1$$ as the charge range and $$t$$ as the discharge duration.

$$\frac{dQ}{Q} = -\frac{dt}{CR}$$

$$\int^{Q_1}_{Q_0}\frac{1}{Q} \, dQ = -\int^{t}_{0s}\frac{1}{CR} \, dt$$

$$\left[\ln Q + k\right]^{Q_1}_{Q_0} = -\frac{1}{CR}\left[t + k\right]^{t}_{0s}$$

$$\ln Q_1 - \ln Q_0 = -\frac{1}{CR} * t$$

$$\ln\frac{Q_1}{Q_0} = -\frac{t}{CR}$$

$$\frac{Q_1}{Q_0} = e^{-\frac{t}{CR}}$$

$$Q_1 = Q_0e^{-\frac{t}{CR}}$$

For the charge we can substitute it with the formula for capacitance $$Q = UC$$. Remember $$Q_0$$ is the charge at the `beginning` of the discharge, so we can substitute it with the capacitor voltage of the beginning or voltage at max charge.

$$U = U_Ce^{-\frac{t}{CR}}$$