---
layout: post
title:  "Steinhart Equation Inverse Derivation and Back"
date:   2020-09-23 22:04:48 +0200
categories: Random-Facts math calculation Temperature Resistance
usemathjax: true
---
The Steinhart Equation models the relationship between the temperature and resistance of a thermistor. It's defined like this:

$$\frac{1}{T} = A + B \ln R + C\ln^3R$$

$$A$$, $$B$$ and $$C$$ are the coefficients used to determine the thermistor.

$$\frac{1}{T} - A = B \ln R + C\ln^3R$$

$$\frac{\frac{1}{T} - A}{C} = \frac{B \ln R}{C} + \ln^3R$$

By using Cardano's method of the cubic equation we can determine $$\ln R$$. To solve the depressed cubic we define the following equations.

$$\frac{B}{C} = 3st \rightarrow s = \frac{B}{3tC}$$

$$\frac{\frac{1}{T} - A}{C} = s^3 - t^3$$

Substituting for $$s$$ makes:

$$\left(\frac{B}{3tC}\right)^3 - t^3 = \frac{\frac{1}{T} - A}{C}$$

$$\frac{B^3}{27t^3C^3} - t^3 = \frac{\frac{1}{T} - A}{C}$$

$$t^6 + t^3 * \frac{\frac{1}{T} - A}{C} - \frac{B^3}{27C^3} = 0$$

Substituting $$t^3 = z$$ allows us to solve the equation.

$$z^2 + z * \frac{\frac{1}{T} - A}{C} + \left(\frac{\frac{1}{T} - A}{2C}\right)^2 - \left(\frac{\frac{1}{T} - A}{2C}\right)^2 - \frac{B^3}{27C^3} = 0$$

$$\left(z + \frac{\frac{1}{T} - A}{2C}\right)^2 = \left(\frac{\frac{1}{T} - A}{2C}\right)^2 + \left(\frac{B}{3C}\right)^3$$

To shorten up we can define $$x = \frac{\frac{1}{T} - A}{2C}$$ and $$y = \sqrt{x^2 + \left(\frac{B}{3C}\right)^3}$$.

$$z = \sqrt{x^2 + \left(\frac{B}{3C}\right)^3} - x = y - x$$

We substitute $$t$$ back in.

$$t = \sqrt[3]{y - x}$$

Now that we have $$t$$ we can substitute it to get $$s$$. We continue using $$s^3 - t^3 = 2x$$.

$$s^3 - (y - x) = 2x$$

$$s^3 = x + y = y + x$$

$$s = \sqrt[3]{y + x}$$

Cardano's method states that $$y = s - t$$. Thus we get the solution:

$$\ln R = \sqrt[3]{y + x} - \sqrt[3]{y - x}$$

$$R = e^{\sqrt[3]{y + x} - \sqrt[3]{y - x}}$$ with $$x = \frac{\frac{1}{T} - A}{2C}$$, $$y = \sqrt{x^2 + \left(\frac{B}{3C}\right)^3}$$

---

To bring back the equation to its original form we define a simpler version:

$$c = a - b$$ with $$a = \sqrt[3]{y + x}$$, $$b = \sqrt[3]{y - x}$$, $$c = \ln R$$

Thus:

$$a^3 = y + x \qquad b^3 = y - x$$

$$a^3b^3 = y^2 - x^2 = (ab)^3$$

$$ab = \sqrt[3]{y^2 - x^2}$$

Expressing $$c$$ as a cubic binominal enables us to get $$\ln R$$.

$$c^3 = (a - b)^3 = a^3 - b^3 + 3ab^2 - 3a^2b = a^3 - b^3 - 3ab(a - b)$$

Notice we can substitute for $$ab$$, $$a - b$$, $$a^3$$ and $$b^3$$ which leads us to the solution.

$$c^3 = y + x - (y - x) - 3\sqrt[3]{y^2 - x^2} * c$$

$$\ln^3R = 2x - 3\sqrt[3]{y^2 - x^2} * \ln R$$

$$\ln^3R = \frac{\frac{1}{T} - A}{C} - 3\sqrt[3]{\left(\sqrt{x^2 + \left(\frac{B}{3C}\right)^3}\right)^2 - x^2} * \ln R$$

$$\ln^3R = \frac{\frac{1}{T} - A}{C} - \frac{B}{C} * \ln R$$

$$C\ln^3R + B\ln R + A = \frac{1}{T}$$