---
title: Bagging techniques
layout: note
---

# Bagging techniques

"Bagging," from the phrase "bootstrap aggregating," means training a bunch of models on different samples of the original training data. We use "bootstrap" samples of the training data, meaning a certain percent is sampled *with replacement*. Due to possible replacement, some instances may be repeated. On each bootstrap sample, a model is trained. All the trained models ultimately have one vote when applied to a new unknown instance. The most common vote wins.

## Benefits

Some machine learning training methods are highly sensitive to variations in the training data. For example, you might get a different decision tree by making small modifications in the training set, or adding or removing some instances. We call such methods "unstable." Other methods, like naïve Bayesian classification, are not as sensitive (they are "stable" methods).

Unstable methods can be made more stable by training several models on different samples of the training data, and then treating each model's prediction on a new instance as a separate "vote." Doing so reduces the variation in the aggregate model's predictions, thus making it more stable.

Some times, applying bagging to an otherwise unstable model, like a decision tree, makes it perform better than a stable model like naïve Bayes.

## Random trees, random forests

Bagging is often used on decision trees. Specifically, bagging is often used on "random trees." A random tree looks at a random subset of the attributes and builds a tree on that subset. A random tree is a decision tree for a limited perspective on the data.

A random tree in isolation is likely not as successful as a decision tree. However, if we apply bagging, we can get a collection of different random trees, known as a *random forest*. Each random tree would contribute one vote to a prediction for a new instance. Each random tree represents a random variation of the training set, and therefore is less likely to overfit as much as a decision tree. A random forest is therefore more stable than a decision tree. Additionally, random forests have proven to be extremely effective classifiers across a wide range of applications.

Weka supports random forest classifiers. It also supports random trees in isolation, as well as bagging. Weka's bagging method allows us to specify any kind of classifier. Interestingly, Weka's random forest classifier is literally the bagging method with random tree classifiers.