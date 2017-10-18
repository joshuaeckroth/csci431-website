---
title: Decision trees
layout: note
---

# Decision trees

A decision tree is much like the game [20 Questions](https://en.wikipedia.org/wiki/Twenty_Questions). The resulting model of a decision tree learning procedure is a tree of questions. The leaves of the tree are predictions. For example, consider this decision tree that predicts survival on the Titanic (from [Wikipedia](https://en.wikipedia.org/wiki/Decision_tree_learning)):

![Decision tree for Titanic survival](/images/decision-tree-titanic.png)

Decision trees are constructed by an iterative procedure that determines which feature should be asked about next, according to the information gain (see below) that results from knowing the value for that feature.

## Decision tree learning

A decision tree can be built using the following iterative procedure:

1. Measure the information gain $IG(a)$ for each attribute $a$ in the data set.
2. Pick the $a$ with the greatest information gain $IG(a)>0$ and make a branch for each possible value $v$ of $a$.
3. For each new branch (where $a=v$ for all possible $v$), repeat this procedure at step (1) with a subset of the current data set where $a=v$.
4. If no attribute has positive information gain or there are no remaining attributes, then we are at a leaf node, and no further choices can be made. Mark the leaf node with the majority class value.

These are the basics of the [C4.5 algorithm](https://en.wikipedia.org/wiki/C4.5_algorithm), developed by Ross Quinlan. The C4.5 algorithm is more advanced than described above; it can handle numeric attributes by checking <, >, <=, and >=; and it can prune the tree according to a threshold value. A pruned tree is more generalized and possibly more useful on new, unseen data.

The C4.5 has been ranked the #1 data mining algorithm (of all time) by attendees at a 2006 conference on data mining ([source PDF](http://www.cs.umd.edu/~samir/498/10Algorithms-08.pdf)). And #8 is k-nearest neighbor, for what it's worth.

## Information gain

We can determine which features are most useful by computing the "information gain" of the feature. **Information gain means: if we knew the value of that feature, how much less surprising would the class be?** If we can completely eliminate surprise, e.g., know perfectly the class, by looking at a certain feature, that feature has maximum information gain. On the other hand, one could imagine a feature that contributes no information: knowing the value of the feature does not narrow the class probability at all. For example, knowing that it is raining in Rome probably tells us nothing about whether it is raining in DeLand. The probablity of rain in DeLand would, presumably, be no different whether or not it is raining in Rome.

Information gain of an attribute $a$ according to a specific class (e.g., "gentry") is defined as:

$$
IG(a) = H(I_\text{all}) - H(a),
$$

where $I_{\text{all}}$ means "all instances" and $H(I)$ is defined as:

$$
\begin{eqnarray}
H(I)&=&\frac{\text{num of instances in } I \text{ with class}}{\text{num of instances in } I} * \left(-\log\left(\frac{\text{num of instances in } I \text{ with class}}{\text{num of instances in } I}\right)\right) \\
&&+\frac{\text{num of instances in } I \text{ without class}}{\text{num of instances in } I} * \left(-\log\left(\frac{\text{num of instances in } I \text{ without class}}{\text{num of instances in } I}\right)\right)
\end{eqnarray}
$$

e.g.,

$$
H(I) = -(P(C) \log P(C) + P(\neg C) \log P(\neg C)) = -\sum_{c\in \{C,\neg C\}} P(c)\log P(c).
$$


and $H(a)$ is defined as:

$$
H(a) = \sum_{v\in \text{vals}(a)} \frac{\text{num instances in } I_{\text{all}} \text{ with }a=v}{\text{num instances in } I_{\text{all}}}*H(\text{instances from } I_{\text{all}} \text{ where }a=v).
$$

e.g.,

$$
H(a) = \sum_{v \in \text{vals}(a)} P(a=v)H(I_{a=v}).
$$

You can think of $H(I)$ as measuring the "information" about the class (e.g., "gentry") in examples in the set $I$. High information means the class is more evenly distributed in the examples. $H(a)$, on the other hand, measures the average information about the class once we filter on attribute $a$. So, the difference, $H(I)-H(a)$, will be positive if the class is more imbalanced after selecting on attribute $a$. We "gain information" if we split at attribute $a$ and then know more precisely which class ("gentry" or "not gentry") is most likely. We gain little information if, after splitting on $a$, the two options ("gentry" and "not gentry") are just as equally likely.

For example, consider this table of data:

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


We want to know which feature, Coat Color or Hat Color, has greater "information gain" with respect to the class Gentry. First, we'll ask about Coat Color:

$$
IG(\text{coat color}) = H(I_\text{all}) - H(\text{coat color}).
$$

We can compute $H(I_\text{all})$ first:

$$
H(I_\text{all})=\frac{3}{8}*\left(-\log\left(\frac{3}{8}\right)\right) + \frac{5}{8}*\left(-\log\left(\frac{5}{8}\right)\right) = 0.662.
$$

Next, $H(\text{coat color})$ is:

$$
H(\text{coat color}) = \sum_{v \in \{\text{black,blue,brown}\}} \frac{\text{num instances with coat color = } v}{8}*H(\text{instances with coat color = }v), 
$$

i.e.,

$$
\begin{eqnarray}
H(\text{coat color}) &=& (\text{black:} (3/8)*((2/3)*(-\log(2/3))+(1/3)*(-\log(1/3))) \\
&& + (\text{blue:}(3/8)*((0/3)*(-\log(0/3))+(3/3)*(-\log(3/3))) \\
&& + (\text{brown:}(2/8)*((1/2)*(-\log(1/2))+(1/2)*(-\log(1/2))) \\
&=& 0.412,
\end{eqnarray}
$$

and subtracting this from $H(I_\text{all})$ we have: $0.250$ for coat color.

On the other hand, $H(\text{hat color})$ comes out to be:

$$
\begin{eqnarray}
H(\text{hat color}) &=& (\text{black:} (4/8)*((2/4)*(-\log(2/4))+(2/4)*(-\log(2/4))) \\
&& + (\text{brown:} (4/8)*((1/4)*(-\log(1/4))+(3/4)*(-\log(3/4))) \\
&=& 0.628,
\end{eqnarray}
$$

resulting in information gain (subtracting from $H(I_\text{all})$): $0.034$ for hat color.

Summarizing our results:

- $IG(\text{coat color}) = 0.250$.
- $IG(\text{hat color}) = 0.034$.

Thus, coat color has greater information gain than hat color. **In other words, Coat Color is a better predictor of Gentry. If we only knew the coat color, we'd be closer to an answer than if we only knew the hat color.**

