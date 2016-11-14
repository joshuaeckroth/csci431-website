---
title: Feature engineering
layout: note

---

# Feature engineering

Sometimes, our training and testing data can be quite a mess. If you feed garbage into a machine learning algorithm, you'll get one of two results:

- **The model might perform very well (unexpectedly!) because it memorizes various anomalies represented in the data.** For example, you might be predicting "spam" based on email text, but all spam examples may have a header, embedded in the email, containing a different spam detector's score. Your machine learning algorithm may notice this score and predict spam only when that score is high.
- **The model might perform very poorly (unexpectedly!) because it gets confused by all the bad data and inconsistent formatting.** For example, you might have an abundance of missing values, values in the wrong columns, truncated values, invalid values that should instead be missing (e.g., -1 to represent a person's age), etc.

A little bit, or more likely a lot of preparation of the data can significantly improve prediction accuracy.

## Workflow

1. Open the data in some tool that shows you all the fields. Excel is likely a good choice here.
2. Figure out which fields can be dropped (e.g., database IDs, description fields), which need to be transformed (e.g., numeric to nominal), and which fields need to be more deeply processed (e.g., extract keywords from a description field).
3. **Write code that performs the requisite transformations.** You will likely need to do the same transformations on new data from the same source. If the transformations are not written as code, they may be forgotten or performed incorrectly, and performing them manually is usually very time consuming.
4. Next, **analyze the data in a tool like Weka to determine the relevant features**. Use a technique like "information gain" (described below in the section "Feature selection"). Consider eliminating the other features that appear to contribute no valuable information.
5. Run a few simple machine learning techniques (e.g., [naïve Bayesian](/notes/naive-bayesian.html) and [k-nearest neighbor](/notes/k-nn.html)) and judge whether they are over-performing or under-performing. **Apply your intuition here. Don't be misled by "great" results, they could be a lie.** Something to keep in mind: generally, scores like 70-80% precision and recall are "**both high and low enough to be believable.**"

## Data cleanup

The first step in feature engineering is cleaning the data. Often data is saved in CSV format, and there are some useful tools to assist data cleanup in a reproducible way (e.g., saved in a script file rather than performed manually), using [csvkit](https://csvkit.readthedocs.io/en/0.9.1/) for manipulating CSV files:

Show column names:

```
csvcut -n file.csv
```

Keep only specific columns:

```
csvcut -c 3,4,5,8 file.csv > out.csv
```

Keep only those rows where a column has a particular regex pattern or specific value:

```
csvgrep -c ColumnName -m mypattern file.csv > out.csv
```

You may ultimately choose to write some code in, e.g., Python, to perform more complicated transformations:

```python
import csv
import sys
import re

# usage: python script.py input.csv output.csv

with open(sys.argv[1]) as csvin:
    with open(sys.argv[2], 'wb') as csvout:
        reader = csv.DictReader(csvin)
        writer = csv.DictWriter(csvout, fieldnames=['field2','field3','field7'])
        writer.writeheader()
        for row in reader:
            # create a new field based on another field
            if int(row['field1']) > 0:
                row['field7'] = 1
            else:
                row['field7'] = 0
            # delete fields we don't want to keep
            del row['field1']
            del row['field4']
            del row['field5']
            del row['field6']
            writer.writerow(row)
```

## Feature selection

After cleaning the data, we may choose to drop some useless features. This can result in faster training (less data to consider) as well as more accurate and generalized models (the models are less likely to "memorize" bogus or useless data that are just anomalies about the training set).

We can determine which features are most useful by computing the "information gain" of the feature. **Information gain means: if we knew the value of that feature, how much less surprising would the class be?** If we can completely eliminate surprise, e.g., know perfectly the class, by looking at a certain feature, that feature has maximum information gain. On the other hand, one could imagine a feature that contributes no information: knowing the value of the feature does not narrow the class probability at all. For example, knowing that it is raining in Rome, Italy probably tells us nothing about whether it is raining in DeLand. The probablity of rain in DeLand would, presumably, be no different whether or not it is raining in Rome.

In Weka, this computation is found in the "Select attributes" tab and the Attribute Evaluator is "InfoGainAttributeEval". It is described as follows:

```
Evaluates the worth of an attribute by measuring the information gain with respect to the class.

InfoGain(Class,Attribute) = H(Class) - H(Class | Attribute).
```

Since we're asking about a specific class (the class to be predicted), and suppose it is a binary class. Then we can drop the "class" variable in the equation and analyze the following simpler form:

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


and $H(a)$ is defined as:

$$
H(a) = \sum_{v\in \text{vals}(a)} \frac{\text{num instances in } I_{\text{all}} \text{ with }a=v}{\text{num instances in } I_{\text{all}}}*H(\text{instances from } I_{\text{all}} \text{ where }a=v).
$$

For example, consider this table of data:

| Coat Color | Hat Color | Gentry? |
| ---------- | --------- | ------- |
| Black      | Black     | Yes     |
| Black      | Brown     | No      |
| Blue       | Black     | No      |
| Blue       | Brown     | No      |
| Brown      | Brown     | No      |
| Brown      | Black     | Yes     |

We want to know which feature, Coat Color or Hat Color, has greater "information gain" with respect to the class Gentry. First, we'll ask about Coat Color:

$$
IG(\text{coat color}) = H(I_\text{all}) - H(\text{coat color}).
$$

We can compute $H(I_\text{all})$ first:

$$
H(I_\text{all})=\frac{2}{6}*\left(-\log\left(\frac{2}{6}\right)\right) + \frac{4}{6}*\left(-\log\left(\frac{4}{6}\right)\right) = 0.918.
$$

Next, $H(\text{coat color})$ is:

$$
H(\text{coat color}) = \sum_{v \in \{\text{black,blue,brown}\}} \frac{\text{num instances with coat color = } v}{6}*H(\text{instances with coat color = }v), 
$$

i.e.,

$$
\begin{eqnarray}
H(\text{coat color}) &=& (\text{black:} (2/6)*((1/2)*(-\log(1/2))+(1/2)*(-\log(1/2))) \\
&& + (\text{blue:}(2/6)*((0/2)*(-\log(0/2))+(2/2)*(-\log(2/2))) \\
&& + (\text{brown:}(2/6)*((1/2)*(-\log(1/2))+(1/2)*(-\log(1/2))) \\
&=& 0.667,
\end{eqnarray}
$$

and subtracting this from $H(I_\text{all})$ we have: $0.251$ for coat color.

On the other hand, $H(\text{hat color})$ comes out to be:

$$
\begin{eqnarray}
H(\text{hat color}) &=& (\text{black:} (3/6)*((2/3)*(-\log(2/3))+(1/3)*(-\log(1/3))) \\
&& + (\text{brown:} (3/6)*((0/3)*(-\log(0/3))+(3/3)*(-\log(3/3))) \\
&=& 0.459,
\end{eqnarray}
$$

resulting in information gain (subtracting from $H(I_\text{all})$): $0.459$ for hat color.

Summarizing our results:

- $IG(\text{coat color}) = 0.251$.
- $IG(\text{hat color}) = 0.459$.

Thus, hat color has greater information gain than coat color. **In other words, Hat Color is a better predictor of Gentry. If we only knew the hat color (black or brown), we'd be closer to an answer than if we only knew the coat color.**

Weka finds the same results:

```
=== Run information ===

Evaluator:    weka.attributeSelection.InfoGainAttributeEval 
Search:       weka.attributeSelection.Ranker -T -1.7976931348623157E308 -N -1
Relation:     information-gain-example
Instances:    6
Attributes:   3
              Coat Color
              Hat Color
              Gentry
Evaluation mode:    evaluate on all training data



=== Attribute Selection on all input data ===

Search Method:
	Attribute ranking.

Attribute Evaluator (supervised, Class (nominal): 3 Gentry):
	Information Gain Ranking Filter

Ranked attributes:
 0.459  2 Hat Color
 0.252  1 Coat Color

Selected attributes: 2,1 : 2
```
