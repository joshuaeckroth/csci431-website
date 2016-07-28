---
title: Text processing
layout: note
---

# Text processing

- high
- level
- points

{% comment %}
The goal is to take a new text document (an article from a website,
perhaps) and choose 1, 2, maybe more categories that describe the
document. These documents and categories relate to artificial
intelligence; the documents are obtained from searching the web with
terms like "artificial intelligence," "robot," and "Turing test."

The 19 categories are: [[http://aaai.org/AITopics/AIOverview][AIOverview]], [[http://aaai.org/AITopics/Agents][Agents]], [[http://aaai.org/AITopics/Applications][Applications]],
[[http://aaai.org/AITopics/CognitiveScience][CognitiveScience]], [[http://aaai.org/AITopics/Education][Education]], [[http://aaai.org/AITopics/Ethics][Ethics]], [[http://aaai.org/AITopics/Games][Games]], [[http://aaai.org/AITopics/History][History]], [[http://aaai.org/AITopics/Interfaces][Interfaces]],
[[http://aaai.org/AITopics/MachineLearning][MachineLearning]], [[http://aaai.org/AITopics/NaturalLanguage][NaturalLanguage]], [[http://aaai.org/AITopics/Philosophy][Philosophy]], [[http://aaai.org/AITopics/Reasoning][Reasoning]],
[[http://aaai.org/AITopics/Representation][Representation]], [[http://aaai.org/AITopics/Robots][Robots]], [[http://aaai.org/AITopics/ScienceFiction][ScienceFiction]], [[http://aaai.org/AITopics/Speech][Speech]], [[http://aaai.org/AITopics/Systems][Systems]], [[http://aaai.org/AITopics/Vision][Vision]]. These
links go to the AITopics website, which provides a news service called
/AI in the News/ that must solve the problem of categorizing news
articles into one or more of these 19 categories.

These notes describe how to do the categorization; obtaining documents
and their content is beyond our scope. Thus, we will utilize a
prepared collection of documents. The zip files linked here provide
about 3000 documents with human-chosen categories. We'll take a
portion of these, say 90%, and train our categorization tools. The
remaining 10% will be used for testing purposes. Which 90% and 10% are
training and testing sets should be set randomly (and differently in
perhaps 5 or 10 different experiments to ensure a categorizer's
performance is robust).

[[./downloads/cse630-ai-articles.zip][Zip file of all AI articles]] --- these documents have the raw original text.

[[./downloads/cse630-ai-articles-stemmed.zip][Zip file of all AI articles (stemmed documents)]] --- these documents
are the same as above, but every word in the text has been "stemmed"
(see below). This is the document set we will be working with.

{% endcomment %}

## Word stemming

> In linguistic morphology and information retrieval, stemming is the
process for reducing inflected (or sometimes derived) words to their
stem, base or root form---generally a written word form. The stem need
not be identical to the morphological root of the word; it is usually
sufficient that related words map to the same stem, even if this stem
is not in itself a valid root.
>
> [...]
>
> A stemmer for English, for example, should identify the string "cats"
(and possibly "catlike", "catty" etc.) as based on the root "cat", and
"stemmer", "stemming", "stemmed" as based on "stem". A stemming
algorithm reduces the words "fishing", "fished", "fish", and "fisher"
to the root word, "fish". ([Wikipedia](http://en.wikipedia.org/wiki/Stemming))

## Feature vectors

A document $j$ is represented as:

$$X_j = (x_{j1}, x_{j2}, \dots, x_{jc}),$$

where $c$ is the count of unique words across all documents in the
database. Thus, all documents have vectors of the same number of
dimensions ($c$ dimensions).

### Binary features

$$x_{ji} = \cases{1 & \text{if $j$ is in $i$} \cr 0 & \text{otherwise}}$$

### Frequency count features

$$x_{ji} = tf_{ji},$$

where $tf_{ji}$ is the frequency (count) of the term in the document.

### Normalized frequency count features

$$x_{ji} = \frac{tf_{ji}}{\sqrt{\sum_{k} tf_{ki}^2}}$$

### tf-idf

The document vectors described above all suffer from over-scoring
common words. A common word like "the" does not give any evidence that
one document is closely related to another one just because they both
contain many instances of that word. So in addition to recording how
often a term appears in a document, we need to divide out some measure
of how common is the term across different documents. A term appearing
in nearly every document is not as relevant as a term appearing in
few. Additionally, if a rare term appears more often in a document, it
is even more indicative of the content of that document.

> Suppose we have a set of English text documents and wish to determine
which document is most relevant to the query "the brown cow." A simple
way to start out is by eliminating documents that do not contain all
three words "the", "brown", and "cow", but this still leaves many
documents. To further distinguish them, we might count the number of
times each term occurs in each document and sum them all together; the
number of times a term occurs in a document is called its term
frequency.
>
> However, because the term "the" is so common, this will tend to
incorrectly emphasize documents which happen to use the word "the"
more frequently, without giving enough weight to the more meaningful
terms "brown" and "cow". The term "the" is not a good keyword to
distinguish relevant and non-relevant documents and terms, unlike the
less common words "brown" and "cow". Hence an inverse document
frequency factor is incorporated which diminishes the weight of terms
that occur very frequently in the collection and increases the weight
of terms that occur rarely. ([Wikipedia](http://en.wikipedia.org/wiki/Tf*idf))

Every dimension (word) in the document vector has the following value:

$$x_{ji} = \log(tf_{ji} + 1) \log(n/df_j + 1),$$

where $j$ is the index of a word, $i$ is the index of a document,
$x\_{ji}$ is the value in position $j$ (for word $j$) in the vector
representing document $i$, $tf\_{ji}$ is the frequency (count) of the
term $j$ in the document $i$, $n$ is the number of documents in the
database, and $df\_j$ is the number of documents in the database that
contain at least one occurrence of the term $j$.

![tf-idf](/images/tfidf-graph.png)

This graph shows the tf-idf value for a particular term in a document
that comes from a database of 1,010 documents. The bottom axis is $df$
(document frequency for that term); the top numbered axis (the "depth"
axis) is $tf$ or (term frequency; the number of times that term
appears in the document under consideration); the right axis is the
tf-idf value.

We see in the graph that an uncommon term ($df$ is low) produces a
higher tf-idf value. However, due to the use of logarithms, the tf-idf
value grows little as the term appears more often in the document. In
all, a term that's appears several times in the document and is
somewhat unique to the document is a strong indicator (of category
membership, of relevance to a query containing that term, etc.).

An example of binary, frequency, and tfidf document vectors can be
found in the [[./doc-vectors.xls][doc-vectors.xls]] spreadsheet.

## k-nearest neighbor

Since we have different ways of representing the document vectors, we
have different "distance" calculations. Below, take $\hat{X} =
(\hat{x}\_0, \dots, \hat{x}\_m)$ to be the document we want to classify,
and $X\_j = (x\_{j0}, \dots, x\_{jn})$ to be a document from the database
($j$ will range so we check the distance to every document in the
database).

### With binary features

$$d = \frac{\sum_{i} \hat{x}_{i} x_{ji}}{\sum_{i} \hat{x}_{i}}$$

The numerator of the fraction gives the number of shared words; the
denominator gives the number of words in the document to be
classified. Thus, $0 \leq d \leq 1$.

### With frequency count features

$$d = \sqrt{\sum_{i} (\hat{x}_{i} - x_{ji})^2}$$

This is just Euclidean distance between the two document
vectors. Thus, $0 \leq d < \infty$.

### With tf-idf

$$d = \sum_{i} \hat{x}_{i} x_{ji},$$

assuming the vectors $\hat{X}$ (the document we are trying to
classify) and $X\_j$ (each document in the database) have been
normalized so that each of their Euclidean distances to the origin
vector (all zeros) is equal to 1.0.

The above is a dot-product of two vectors. Because these vectors have
been normalized, the dot-product can also be called the cosine
similarity:

$$\cos \theta = \hat{X} \cdot X_j,$$

where $0 \leq \theta \leq \pi/2$ and $\theta$ is the angle (from the
origin) between the two document vectors in the higher-dimensional
space. More similar documents have a smaller angle between them,
meaning their cosine similarity is closer to 1.0. Documents with
absolutely no terms in common have a dot-product equal to 0.0, or
equivalently, a 90-degree angle between them.

### Comparing k-nearest neighbor solutions

![Comparison](/images/classification-alg-comparison.png)

## Take-away lessons

- Good choices for what I am calling the "algorithm" (tf-idf, count
  with normalization, etc.) depends on the dataset. However, *tf-idf
  is robust* and works best on all datasets represented here. This
  is a significant result.

- Using a weighted voting technique is generally a good idea.

- Increasing $k$ does not always increase performance.

- The best value for $k$ depends on the the dataset and the
  algorithm.

