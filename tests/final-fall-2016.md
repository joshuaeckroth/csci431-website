---
layout: note
title: Final exam for Fall 2016
---

# Final exam for Fall 2016

Submit your solutions on bitbucket in a repository named csci431-final or similar. Create a folder for each question. **Answers are due by Thur Dec 15, 11:59pm.** You may only look at these course notes, any linked page, and standard documentation for ProbLog and Weka. The exam is out of 100 points (but weighted 20% of your overall grade).

## Question 1: ProbLog (20 pts)

Code the following problem and perform queries to evaluate the question stated at the end. Use `evidence()` and `query()` in ProbLog as appropriate.

> My car won't start. It doesn't seem to crank when I turn the key. It's my understanding that starting a car works as follows. If the battery is good and the alternator works, the battery will have sufficient voltage (always). But if the battery is good but the alternator is not ok, there is a 60% chance the battery has sufficient voltage. The spark plugs will spark (always) if the battery has sufficient voltage and the spark plugs are ok. The starter system will cause the car to crank if there is sufficient voltage. Cranking plus sparks plus a working fuel system will almost always (90% of the time) cause the car to start. The other 10% of the time accounts for all the other possible causes that aren't elucidated here. Additionally, sufficient battery voltage plus good headlight bulbs will cause the headlights to turn on, but they don't in fact. I'm 80% confident the headlight bulbs are working, 90% confident the battery is ok since it's only a couple months old, and entirely unsure about the alternator, fuel system, and spark plugs (i.e., 50% sure that each are in working order). The question is, is the battery bad or the spark plugs?

You should arrive at the following conclusions:

```
   battery_ok:  0.74463739
sparkplugs_ok:  0.45863126
```

## Question 2: Classification evaluation (5 pts)

### Part 1: Manual calculations (3/5 pts)

Given the following "ground truth" and predicted values, calculate precision, recall, and F-score, and write the confusion matrix. The values are given in the same order, e.g., case 1, case 2, …, case n.

```
Groundtruth: [negative, negative, positive, negative, positive, positive, negative]
Predicted:   [negative, positive, positive, negative, positive, positive, positive]
```

### Part 2: Weka (2/5 pts)

Refer to the Weka output below (on the Iris dataset) to answer the following questions:

- Which Iris species is the most "confused" by this classifier? Also, when the classifier misclassifies that species, which species does it usually misclassify it as?
- What does it mean to say the classifier has perfect precision and recall on Iris-setosa, as shown in the output? Don't tell us the numbers; we can see they are 1.0 for precision and recall. Instead, describe what we should expect from the classifier, with respect to Iris-setosa, when we give it new flower measurements.

```
=== Summary ===

Correctly Classified Instances         137               91.3333 %
Incorrectly Classified Instances        13                8.6667 %
Kappa statistic                          0.87  
Mean absolute error                      0.1642
Root mean squared error                  0.2331
Relative absolute error                 36.9555 %
Root relative squared error             49.4534 %
Total Number of Instances              150     

=== Detailed Accuracy By Class ===

                 TP Rate  FP Rate  Precision  Recall   F-Measure  MCC      ROC Area  PRC Area  Class
                 1.000    0.000    1.000      1.000    1.000      1.000    1.000     1.000     Iris-setosa
                 0.960    0.110    0.814      0.960    0.881      0.820    0.980     0.956     Iris-versicolor
                 0.780    0.020    0.951      0.780    0.857      0.804    0.981     0.963     Iris-virginica
Weighted Avg.    0.913    0.043    0.922      0.913    0.913      0.875    0.987     0.973     

=== Confusion Matrix ===

  a  b  c   <-- classified as
 50  0  0 |  a = Iris-setosa
  0 48  2 |  b = Iris-versicolor
  0 11 39 |  c = Iris-virginica
```

## Question 3: Feature engineering (20 pts)

### Part 1: Data cleanup (15/20 pts)

Grab the [Orlando Crime Data](https://data.cityoforlando.net/Orlando-Police/City-Of-Orlando-Crimes/hm2t-fd4m). Produce a Weka ARFF file that is appropriate for predicting the Case Offense Category (Assault, Theft, Robbery, etc.) based on the hour of the day (0-23), street name (choose first street name if the crime took place at an intersection), latitude, longitude (separated from latitude), and location type (Residence/Single, Bar/Nightclub, etc.). Use appropriate data types in Weka (nominal, numeric, etc.), remove any rows that do not have the correct format or have missing data, and remove any columns/attributes that contain information that shouldn't be exposed to the classifier or are otherwise irrelevant.

Also report the steps that you took in your clean up process, or submit code that performs the operations.

### Part 2: Information gain (5/20 pts)

Take the [Iris dataset as an ARFF file](/iris.arff) and determine the top two most "useful" attributes in terms of information gain. Specify which attribute has highest information gain and which has second-highest.

## Question 4: K-nearest neighbor (10 pts)

Take the [Iris dataset as an ARFF file](/iris.arff) and determine the best value of K (i.e., number of neighbors) for predicting iris species. Do not enable distance weighting. (Note, some different values of K yield the same highest accuracy; just give one such value.)

## Question 5: Decision trees (10 pts)

Take the [mammographic masses](/mammographic_masses.csv) dataset (as a CSV file) and load into Weka. Remove the "birad" column since the [description of the dataset](http://archive.ics.uci.edu/ml/datasets/Mammographic+Mass) says it's non-predictive. Next change "shape," "margin," and "severity" to nominals, as the description states.

Using Weka, "learn" a decision tree that has fewer than 10 leaves ("leaves" as reported by Weka). Report the classifier's accuracy after 10-fold cross validation. Then describe this tree in English sentences so a non-expert can follow the logic and arrive at a conclusion.

## Question 6: Naïve Bayesian classification (15 pts)

### Part 1: Manual calculations (10/15 pts)

Referring back to the [Bayesian inference notes](/notes/bayesian-inference.html), show all the math for computing the answer to the following scenario. Write the formula you are calculating above each calculation. Make up your own numbers for prior and conditional probabilities.

> Suppose your friend attends Big State School and aced her computer science class. You can imagine two possible reasons for this: your friend either studied very hard and/or the class was very easy. If you are forced to choose only one of those options as the explanation, which option is more plausible?

### Part 2: Weka (5/15 pts)

Take the [Iris dataset as an ARFF file](/iris.arff) and train a Naïve Bayes classifier. Copy-paste all Weka output for 10-fold cross validation.

## Question 7: Text processing (15 pts)

Take this [Yelp review dataset](/yelp_labelled.csv) (from [UCI again](http://archive.ics.uci.edu/ml/datasets/Sentiment+Labelled+Sentences)) and process the first attribute containing restaurant review comments. (Note, the second column is 1 or 0 depending on whether the review is positive or negative.) (Also note, some reviews have offensive language.)

Transform the first attribute into distinct words. Keep the top 1000 words, lowercase all words, and stem all words. The resulting columns should contain word counts, not just binary values. Use normal English tokenization rules (or change the tokenization rules if you wish). Make sure the class column (Score) is the last attribute after processing. Submit the resulting ARFF file.

## Question 8: Multilayer perceptrons (5 pts)

Take the [Iris dataset as an ARFF file](/iris.arff) and train an MLP with two hidden layers: first with 10 neurons, second with 5 neurons. Weka will decide the size of the inputs as well as the final output layer. Use a learning rate of 0.1 and 200 epochs. Copy-paste all Weka output for 10-fold cross validation.