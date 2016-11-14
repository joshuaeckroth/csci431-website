---
title: Decision trees
layout: note
---

# Decision trees

A decision tree is much like the game [20 Questions](https://en.wikipedia.org/wiki/Twenty_Questions). The resulting model of a decision tree learning procedure is a tree of questions. The leaves of the tree are predictions. For example, consider this decision tree that predicts survival on the Titanic (from [Wikipedia](https://en.wikipedia.org/wiki/Decision_tree_learning)):

![Decision tree for Titanic survival](/images/decision-tree-titanic.png)

Decision trees are constructed by an iterative procedure that determines which feature should be asked about next, according to the [information gain](/notes/feature-engineering.html#tocAnchor-1-3) that results from knowing the value for that feature.

## Decision tree learning

A decision tree can be built using the following iterative procedure:

1. Measure the information gain $IG(a)$ for each attribute $a$ in the data set.
2. Pick the $a$ with the greatest information gain $IG(a)>0$ and make a branch for each possible value $v$ of $a$.
3. For each new branch (where $a=v$ for all possible $v$), repeat this procedure at step (1) with a subset of the current data set where $a=v$.
4. If no attribute has positive information gain or there are no remaining attributes, then we are at a leaf node, and no further choices can be made. Mark the leaf node with the majority class value.

These are the basics of the [C4.5 algorithm](https://en.wikipedia.org/wiki/C4.5_algorithm), developed by Ross Quinlan. The C4.5 algorithm is more advanced than described above; it can handle numeric attributes by checking <, >, <=, and >=; and it can prune the tree according to a threshold value. A pruned tree is more generalized and possibly more useful on new, unseen data.

The C4.5 has been ranked the #1 data mining algorithm (of all time) by attendees at a 2006 conference on data mining ([source PDF](http://www.cs.umd.edu/~samir/498/10Algorithms-08.pdf)). And #8 is [k-nearest neighbor](/notes/k-nn.html), for what it's worth.

## Simple example

Consider the gentry dataset from the [feature engineering notes](/notes/feature-engineering.html):

| Coat Color | Hat Color | Gentry? |
| ---------- | --------- | ------- |
| Black      | Black     | Yes     |
| Black      | Brown     | No      |
| Blue       | Black     | No      |
| Blue       | Brown     | No      |
| Brown      | Brown     | No      |
| Brown      | Black     | Yes     |

We previously computed the information gain of the two possible attributes, coat color and hat color:

- $IG(\text{coat color}) = 0.251$.
- $IG(\text{hat color}) = 0.459$.

Thus, the first split will be on hat color. Now we have a root question ("what is the hat color?") and two answers: black or brown.

For the black hat subtree, the next question (the only remaining question) is: what is the coat color? There are two possible answers, and thus two branches. Both branches are leaves because there are no other possible questions. For branch "coat color = black", the sole leaf has "gentry = yes"; for "coat color = blue", the sole leaf has "gentry = no"; and for "coat color = brown", the sole leaf has "gentry = yes".

For the brown hat subtree, there is no next question because "coat color" has zero information gain. For every coat color, the answer is "gentry = no".

## Decision trees in Weka

Weka has a category of classifiers called "trees." The one we want is J48, an open-source version of C4.5. We can force no pruning and allow leaf nodes to contain only a single example by setting "unpruned = True" and "minNumObj = 1". Here is the resulting tree for the gentry data:

```
J48 unpruned tree
------------------

Hat Color = Black
|   Coat Color = Black: Yes (1.0)
|   Coat Color = Blue: No (1.0)
|   Coat Color = Brown: Yes (1.0)
Hat Color = Brown: No (3.0)

Number of Leaves  : 	4

Size of the tree : 	6
```

Here is Weka's visualization of the tree:

![Weka decision tree](/images/weka-decision-tree-gentry.png)

Advice for J48:

- lower the "confidenceFactor" parameter to increase pruning; a pruned tree is a more general tree and might perform better on test data
- increase the "minNumObj" parameter to prevent too many branches for less common situations; again, a high value in parameter produces smaller trees that are more general