# Algebra, Functions, and Notation

## Learning objectives

After this lesson, you should be able to read common mathematical notation in an ML paper, substitute values into a function, and keep track of what a symbol represents.

## 1. Variables and functions

A **variable** is a name for a value. A **function** takes an input and returns an output. For example, if a taxi fare is a fixed fee plus a charge per kilometer, we can write

$$
f(x) = 3 + 2x,
$$

where $x$ is the distance in kilometers and $f(x)$ is the fare. At 4 km, $f(4)=3+2(4)=11$.

In ML, a model is also a function. It takes features $x$ and produces a prediction $f(x)$. Training changes parameters, often written as $w$ or $\theta$, so predictions better match examples.

## 2. Equations and rearranging

An equation says two expressions have the same value. To solve $2x+3=11$, undo operations in reverse order: subtract 3, then divide by 2, so $x=4$. The same operation must be applied to both sides.

This matters when interpreting formulas. If $\hat y=wx+b$, then the prediction $\hat y$ changes with input $x$; $w$ controls slope and $b$ is the value when $x=0$.

## 3. Powers, roots, and logarithms

- $x^2$ means $x\cdot x$; $\sqrt{x}$ is the nonnegative number whose square is $x$.
- $a^b$ asks how many times to multiply $a$ by itself when $b$ is a positive integer. Exponent rules extend this idea.
- $\log(x)$ reverses exponentiation. With the natural logarithm, $\ln(e^x)=x$.

Logarithms turn multiplication into addition: $\ln(ab)=\ln(a)+\ln(b)$. In machine learning this is useful for probabilities, whose products can become tiny. We often maximize log probability instead of probability because it is numerically easier and has the same maximizer.

## 4. Sums and averages

The symbol $\sum_{i=1}^{n}x_i$ means add $x_1+x_2+\cdots+x_n$. The average is

$$
\bar{x} = \frac{1}{n}\sum_{i=1}^{n} x_i.
$$

For values $2,4,9$, the sum is 15 and the mean is 5. Many training objectives are averages of per-example losses:

$$
J(\theta)=\frac{1}{n}\sum_{i=1}^{n}\ell_i(\theta).
$$

## 5. Shapes, units, and subscripts

Notation is a compact language. A subscript often identifies an example: $x_i$ is example $i$. A superscript may identify a power ($x^2$) or a layer ($h^{(2)}$); check the author’s definition. A vector with 3 entries has shape `(3,)`; a matrix with 5 rows and 3 columns has shape `(5, 3)`.

Attach meaning and units to values. If a feature is in centimeters but a formula expects meters, a numerical result can be wrong by a factor of 100. In code, variable names such as `distance_m` can prevent that mistake.

## Worked example

A model predicts house price with $p=50+4a$, where $a$ is area measured in tens of square meters and $p$ is in thousands of dollars. For a 70-square-meter home, $a=7$, so $p=50+4(7)=78$ thousand dollars. The model's slope says the prediction rises by 4 thousand dollars per additional 10 square meters. It does **not** prove that adding area causes that price increase; the model only describes its fitted relationship.

## Check your understanding

1. If $g(x)=5x-2$, what is $g(3)$?
2. Write the average of five values using a summation.
3. A dataset has 100 examples and 8 features. What shape might its feature matrix have?

**Answers:** 13; $\frac15\sum_{i=1}^{5}x_i$; `(100, 8)` if rows are examples and columns are features.

## Common mistakes

- Treating a variable name as if it had a fixed meaning across every book or paper.
- Forgetting whether a vector is a row or column vector.
- Confusing $\log(x^2)$ with $(\log x)^2$.
- Ignoring units or the number of examples represented by an average.
