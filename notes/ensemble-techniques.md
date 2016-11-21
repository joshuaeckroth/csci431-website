---
title: Ensemble techniques
layout: note
---

# Ensemble techniques

Ensemble learning techniques combine multiple models into an aggregate model. Each of the models in the aggregate acts as a "hypothesis" when predicting the class for a new instance. Each model should be different in some way: either trained on different data (e.g., bagging or boosting), trained with different parameters (e.g., random trees), or altogether a different kind of model (e.g., combining naïve Bayes with k-nearest neighbor). Often, the predictions produced by the various models are treated as votes, and the most common prediction wins.

All of these ensemble techniques may be found in Weka's "meta" category of classifiers.

## Voting

The process of voting is straightforward: take various models, each trained on either the same data or different subsets of the data, and count each model's prediction as one vote. Then, find the most common vote (the "mode"). That's the winning prediction for the ensemble.

## Boosting

Boosting is the process of combining "weak" models (like really small decision trees, called "decision stumps") into an aggregate strong model. Each of the weak models performs little better than random guessing, but when combined perform very well. AdaBoost ("adaptive boosting") is a common procedure for building the ensemble model. The procedure works as follows:

1. Train a weak model, e.g., a "decision stump" (one-level decision tree), on the original data.
2. Test the weak model on the training data. Collect all the misclassifications (errors).
3. Add positive weights to the misclassified instances. The weak model should make use of these weights to prioritize what it learns (e.g. the decision stump should focus on predicting the classes of the higher-weighted instances).
4. Go back to step 1.

Collect all the resulting weak models, each trained on the weighted errors of the prior models. Rather than strict voting, where every vote has the same influence, add weights to the votes that correspond to the accuracy of each model in the aggregate.

## Bagging

"Bagging," from the phrase "bootstrap aggregating," means training a bunch of models on different samples of the original training data. We use "bootstrap" samples of the training data, meaning a certain percent is sampled *with replacement*. Due to possible replacement, some instances may be repeated. On each bootstrap sample, a model is trained. All the trained models ultimately have one vote when applied to a new unknown instance. The most common vote wins.

### Benefits

Some machine learning training methods are highly sensitive to variations in the training data. For example, you might get a different decision tree by making small modifications in the training set, or adding or removing some instances. We call such methods "unstable." Other methods, like naïve Bayesian classification, are not as sensitive (they are "stable" methods).

Unstable methods can be made more stable by training several models on different samples of the training data, and then treating each model's prediction on a new instance as a separate "vote." Doing so reduces the variation in the aggregate model's predictions, thus making it more stable.

Some times, applying bagging to an otherwise unstable model, like a decision tree, makes it perform better than a stable model like naïve Bayes.

### Random trees, random forests

Bagging is often used on decision trees. Specifically, bagging is often used on "random trees." A random tree looks at a random subset of the attributes and builds a tree on that subset. A random tree is a decision tree for a limited perspective on the data.

A random tree in isolation is likely not as successful as a decision tree. However, if we apply bagging, we can get a collection of different random trees, known as a *random forest*. Each random tree would contribute one vote to a prediction for a new instance. Each random tree represents a random variation of the training set, and therefore is less likely to overfit as much as a decision tree. A random forest is therefore more stable than a decision tree. Additionally, random forests have proven to be extremely effective classifiers across a wide range of applications.

Weka supports random forest classifiers. It also supports random trees in isolation, as well as bagging. Weka's bagging method allows us to specify any kind of classifier. Interestingly, Weka's random forest classifier is literally the bagging method with random tree classifiers.

## Stacking

After reviewing all of these ensemble techniques, a clever person may naturally ask, "What if we train a model on the outputs of various other models?" This is known as "stacking."

Stacking works exactly as you would expect. In Weka, we can select all the submodels (e.g., random forests, naïve Bayes, k-nearest neighbor, etc.) plus the "meta" model that trains on the output of the submodels. The meta model can be any arbitrary kind, e.g., naïve Bayes.