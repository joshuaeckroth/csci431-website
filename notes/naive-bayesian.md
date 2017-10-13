---
title: Naïve Bayesian inference
layout: note
---

# Naïve Bayesian inference

Recall from the [Bayesian inference notes](/notes/bayesian-inference.html) that Bayes' theorem allows us to take a known probabilistic relationship, such as $P(\text{toothache}\|\text{gum disease},\neg\text{cavity}) = 0.3$, and reason backwards. For example, we can determine $P(\text{gum disease}\|\text{toothache})$, read "the probability that the patient has gum disease given that they have a toothache," assuming we have knowledge of the various "conditionals" such as $P(\text{toothache}\|\text{gum disease},\text{cavity})$ et al., and "priors" $P(\text{gum disease})$, $P(\text{cavity})$.

We can call "toothache" the *evidence* and "gum disease" the *prediction*.  Naturally, we want to *learn* the probabilities from training data rather than code them by hand. Doing so efficiently and in a way that yields an efficient Bayesian inference procedure requires that we make the "naïve assumption."

## Without the naïve assumption

Suppose we have some training examples:

| Coat Color | Hat Color | Gentry? |
| ---------- | --------- | ------- |
| Black      | Black     | Yes     |
| Black      | Black     | No      |
| Black      | Brown     | Yes     |
| Blue       | Black     | No      |
| Blue       | Brown     | No      |
| Blue       | Brown     | No      |
| Brown      | Black     | Yes     |
| Brown      | Brown     | No      |

Our predicted column will be Gentry and the evidence columns will be Coat and Hat Color. Bayesian inference treats this as a probabilistic inference problem: $P(\text{Gentry}\|\text{Coat=Black}, \text{Hat=Brown}) = ?$.

In order to solve that probability, we need to apply Bayes' rule:



$$
P(\text{Gentry}|\text{Coat=Black},\text{Hat=Brown}) = \frac{P(\text{Coat=Black},\text{Hat=Brown}|\text{Gentry})P(\text{Gentry})}{P(\text{Coat=Black},\text{Hat=Brown})}.
$$



So far so good. We can easily learn $P(\text{Gentry})$ from the data:



$$
P(\text{Gentry}) = \frac{\text{# of training instances with Gentry = Yes}}{\text{# of training instances}} = 3/8 = 0.375.
$$


But $P(\text{Coat=Black},\text{Hat=Brown}\|\text{Gentry})$ and $P(\text{Coat=Black},\text{Hat=Brown})$ start to spell trouble. In this simple example, they are easy to compute. We can just count the number of rows in the table. But notice that $P(\text{Coat},\text{Hat}\|\text{Gentry})$ represents a table containing $3\*2\*2=12$ probabilities for all the variations of coat, hat, and gentry. Consider the table for Gentry=Yes (the 3 is due to having 3 cases of Gentry=Yes):

| Coat Color | Hat Color | $P(\text{Coat},\text{Hat}\|\text{Gentry=Yes})$ |
| ---------- | --------- | ---------------------------------------- |
| Black      | Black     | 1/3 = 0.33                               |
| Black      | Brown     | 1/3 = 0.33                               |
| Blue       | Black     | 0/3 = 0.0                                |
| Blue       | Brown     | 0/3 = 0.0                                |
| Brown      | Black     | 1/3 = 0.33                               |
| Brown      | Brown     | 0/3 = 0.0                                |

We also have a table for the probability of Coat and Hat Color co-occurring (the 8 is due to having 8 training examples in total):

| Coat Color | Hat Color | $P(\text{Coat},\text{Hat})$ |
| ---------- | --------- | --------------------------- |
| Black      | Black     | 2/8 = 0.25                  |
| Black      | Brown     | 1/8 = 0.125                 |
| Blue       | Black     | 1/8 = 0.125                 |
| Blue       | Brown     | 2/8 = 0.25                  |
| Brown      | Black     | 1/8 = 0.125                 |
| Brown      | Brown     | 1/8 = 0.125                 |

**Now imagine we had $n$ different attributes**, ignoring the predicted attribute. Bayes' rule gives us:



$$
P(\text{pred}|a_1,a_2,\dots,a_n)=\frac{P(a_1,a_2,\dots,a_n|\text{pred})P(\text{pred})}{P(a_1,a_2,\dots,a_n)}.
$$



The tables for $P(a\_1,a\_2,\dots,a\_n\|\text{pred})$ and $P(a\_1,a\_2,\dots,a\_n)$ will have $2^n$ rows for each pred, assuming binary attributes. In other words, a simplistic Bayesian approach to the problem yields exponential models. That is unacceptable.

## With the naïve assumption

In order to reduce the complexity of the model, we'll make a "naïve assumption." We'll assume all the evidence attributes (e.g., Coat Color and Hat Color) are **independent events**. This is probably not true at all: most gentry will intentionally coordinate their coat and hat color choices. But we make this assumption for expedience, even if it does not match reality.

By making this assumption, we transform the exponential model into a linear model:



$$
P(\text{pred}|a_1,a_2,\dots,a_n)=\frac{P(a_1|\text{pred})P(a_2|\text{pred})\cdots P(a_n|\text{pred})P(\text{pred})}{P(a_1)P(a_2)\cdots P(a_n)},
$$


or stated more compactly,



$$
P(\text{pred}|a_1,a_2,\dots,a_n)=\frac{\prod_i P(a_i|\text{pred}) P(\text{pred})}{\prod_i P(a_i)}.
$$



Now our probability tables are very small. First, the probability of a coat color if you know the gentry status:

| Coat Color | $P(\text{Coat}\|\text{Gentry=Yes})$ |
| ---------- | ----------------------------------- |
| Black      | 2/3 = 0.66                          |
| Blue       | 0/3 = 0.0                           |
| Brown      | 1/3 = 0.33                          |

Now the probability of hat color if you know the gentry status:

| Hat Color | $P(\text{Hat}\|\text{Gentry=Yes})$ |
| --------- | ----------------------------------- |
| Black     | 2/3 = 0.66                          |
| Brown     | 1/3 = 0.33                          |

Now the "prior" probability of coat color:

| Coat Color | $P(\text{Coat})$ |
| ---------- | ---------------- |
| Black      | 3/8 = 0.375      |
| Blue       | 3/8 = 0.375      |
| Brown      | 2/8 = 0.25       |

And the "prior" probability of hat color:

| Hat Color | $P(\text{Hat})$ |
| --------- | --------------- |
| Black     | 4/8 = 0.5       |
| Brown     | 4/8 = 0.5       |

The resulting Bayesian network, with the naïve assumption in place, looks like this:

![Gentry Bayesian network](/images/gentry.png)

## Gentry example in ProbLog

ProbLog can learn probabilities from data. Rather than add a number in front of a predicate, e.g., `0.5::coat(black)`, we can indicate the probability is "tunable" with `t()`: `t(_)::coat(black)` or we can start with an initial probability that will update from training: `t(0.5)::coat(black)`.

Here is our domain logic:

```
coatColor(black).
coatColor(blue).
coatColor(brown).

hatColor(black).
hatColor(brown).

t(_, CoatColor)::coat(CoatColor) :- coatColor(CoatColor).
t(_, HatColor)::hat(HatColor) :- hatColor(HatColor).

t(_, CoatColor, HatColor)::gentry :-
    coatColor(CoatColor), hatColor(HatColor),
    coat(CoatColor), hat(HatColor).
```

And here is our evidence file, with different cases separated by dashes:

```
evidence(gentry).
evidence(coat(black)).
evidence(coat(blue), false).
evidence(coat(brown), false).
evidence(hat(black)).
evidence(hat(brown), false).
evidence(hat(blue), false).
-----
evidence(gentry, false).
evidence(coat(black)).
evidence(coat(blue), false).
evidence(coat(brown), false).
evidence(hat(black)).
evidence(hat(brown), false).
evidence(hat(blue), false).
-----
evidence(gentry).
evidence(coat(black)).
evidence(coat(blue), false).
evidence(coat(brown), false).
evidence(hat(brown)).
evidence(hat(black), false).
evidence(hat(blue), false).
-----
evidence(gentry, false).
evidence(coat(blue)).
evidence(coat(black), false).
evidence(coat(brown), false).
evidence(hat(black)).
evidence(hat(blue), false).
evidence(hat(brown), false).
-----
evidence(gentry, false).
evidence(coat(blue)).
evidence(coat(black), false).
evidence(coat(brown), false).
evidence(hat(brown)).
evidence(hat(black), false).
evidence(hat(blue), false).
-----
evidence(gentry, false).
evidence(coat(blue)).
evidence(coat(black), false).
evidence(coat(brown), false).
evidence(hat(brown)).
evidence(hat(black), false).
evidence(hat(blue), false).
-----
evidence(gentry).
evidence(coat(brown)).
evidence(coat(black), false).
evidence(coat(blue), false).
evidence(hat(black)).
evidence(hat(brown), false).
evidence(hat(blue), false).
-----
evidence(gentry, false).
evidence(coat(brown)).
evidence(coat(black), false).
evidence(coat(blue), false).
evidence(hat(brown)).
evidence(hat(black), false).
evidence(hat(blue), false).
```

Now we run the ProbLog command to learn the tunable probabilities from the evidence:

```
problog lfi -O gentry.model gentry.pl gentry-evidence.pl
```

The resulting model is:

```
coatColor(black).
coatColor(blue).
coatColor(brown).
hatColor(black).
hatColor(brown).
0.375::coat(blue) :- coatColor(blue).
0.25::coat(brown) :- coatColor(brown).
0.375::coat(black) :- coatColor(black).
0.5::hat(black) :- hatColor(black).
0.5::hat(brown) :- hatColor(brown).
0.0::gentry :- coatColor(brown), hatColor(brown), coat(brown), hat(brown).
0.0::gentry :- coatColor(blue), hatColor(brown), coat(blue), hat(brown).
0.0::gentry :- coatColor(blue), hatColor(black), coat(blue), hat(black).
0.999999999702::gentry :- coatColor(black), hatColor(brown), coat(black), hat(brown).
0.5::gentry :- coatColor(black), hatColor(black), coat(black), hat(black).
0.999999999702::gentry :- coatColor(brown), hatColor(black), coat(brown), hat(black).
```

## A larger example

Consider the Iris dataset. In this case, the four attributes are continuous rather than discrete, so we'll model them with Gaussian distributions. Thus, we need to compute the mean and standard deviation for each attribute, both as priors (across all the instances) and conditionals (for each subset of iris species).

| Class (or "Prior") | Attribute    | Mean, standard deviation |
| ------------------ | ------------ | ------------------------ |
| (Prior)            | Sepal length | m=5.843, sd=0.825        |
| (Prior)            | Sepal width  | m=3.054, sd=0.432        |
| (Prior)            | Petal length | m=3.759, sd=1.759        |
| (Prior)            | Petal width  | m=1.199, sd=0.761        |
| Iris-setosa        | Sepal length | m=5.006, sd=0.349        |
| Iris-setosa        | Sepal width  | m=3.418, sd=0.377        |
| Iris-setosa        | Petal length | m=1.464, sd=0.172        |
| Iris-setosa        | Petal width  | m=0.244, sd=0.106        |
| Iris-versicolor    | Sepal length | m=5.936, sd=0.511        |
| Iris-versicolor    | Sepal width  | m=2.770, sd=0.311        |
| Iris-versicolor    | Petal length | m=4.260, sd=0.465        |
| Iris-versicolor    | Petal width  | m=1.326, sd=0.196        |
| Iris-virginica     | Sepal length | m=6.588, sd=0.629        |
| Iris-virginica     | Sepal width  | m=2.974, sd=0.319        |
| Iris-virginica     | Petal length | m=5.552, sd=0.546        |
| Iris-virginica     | Petal width  | m=2.026, sd=0.272        |

We can compute probability densities using the mean and standard deviation with the "probability density function," known as `dnorm` in the R language, and denoted $P_D$ in the calculations below. The probability density function gives the y-axis value of a plot of the probability distribution. A probability density is not a probability, but we can get probabilities at the end by normalizing (dividing each result, as a density, by the sum of densities).

The resulting Bayesian network representing the relationships of the attributes and class (species) is shown below. This network illustrates the "naïve assumption," namely that the attributes are independent of each other (there are no arrows between attributes).

![Iris Bayesian network](/images/iris-bayes.png)

Now, suppose we have measurements of a new flower:

| Sepal length | Sepal width | Petal length | Petal width |
| ------------ | ----------- | ------------ | ----------- |
| 5.8          | 2.8         | 4.0          | 1.4         |

We can perform naïve Bayesian classification by computing the probability of each class, given these measurements. Note, it's pretty clearly an example of Iris-versicolor.

First, the probability of Iris-setosa (IS):


$$
\begin{eqnarray}
P(\text{IS}|\text{measurements})&\simeq&P_D(S_L=5.8|\text{IS})P_D(S_W=2.8|\text{IS})P_D(P_L=4.0|\text{IS})P_D(P_W=1.4|\text{IS})P(\text{IS})\\
&&/(P_D(S_L=5.8)P_D(S_W=2.8)P_D(P_L=4.0)P_D(P_W=1.4))\\
&=&0.086*0.276*0.000*0.000*(1/3)/(0.483*0.777*0.225*0.506)\\
&=& 0.000/0.043\\
&=& 0.000.
\end{eqnarray}
$$


Due to the much larger petal length and width than is typical for Iris-setosa, the probability density is virtually 0.0. Now, consider Iris-versicolor:


$$
\begin{eqnarray}
P(\text{IVe}|\text{measurements})&\simeq&P(S_L=5.8|\text{IVe})P(S_W=2.8|\text{IVe})P(P_L=4.0|\text{IVe})P(P_W=1.4|\text{IVe})P(\text{IVe})\\
&&/(P(S_L=5.8)P(S_W=2.8)P(P_L=4.0)P(P_W=1.4))\\
&=&1.059*1.277*0.734*1.895*(1/3)/0.043\\
&=& 14.436.
\end{eqnarray}
$$


This result is >1 because we are using $P_D$, the probability density function, which is not the same as a probability. But we can normalize these results at the end to get actual probabilities.

Finally, we'll check on Iris-virginica:


$$
\begin{eqnarray}
P(\text{IVi}|\text{measurements})&\simeq&P(S_L=5.8|\text{IVi})P(S_W=2.8|\text{IVi})P(P_L=4.0|\text{IVi})P(P_W=1.4|\text{IVi})P(\text{IVi})\\
&&/(P(S_L=5.8)P(S_W=2.8)P(P_L=4.0)P(P_W=1.4))\\
&=&0.289*1.078*0.013*0.104*(1/3)/0.043\\
&=& 0.003.
\end{eqnarray}
$$


We can already see that Iris-versicolor got the most votes. To get actual probabilities, we would divide each result by the sum, giving Iris-versicolor a probability near 1.0.
