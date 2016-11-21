---
title: Weka for classification
layout: note
---

# Weka for classification

Download Weka from the [Weka website](http://www.cs.waikato.ac.nz/ml/weka/downloading.html), and download the "stable" version for your machine.

## GUI

Click "Choose" to choose the classification algorithm.

![Weka](/images/weka-classify-1.png)

Choose "IBk."

![Weka](/images/weka-classify-2.png)

Click the name of the algorithm to get more options.

![Weka](/images/weka-classify-3.png)

Possibly change the number of clusters.

![Weka](/images/weka-classify-4.png)

For evaluation (including the confusion matrix), choose "Cross-validation" with 10 folds.

![Weka](/images/weka-classify-5.png)

## Command-line

The following examples assume you are working on delenn. You can download Weka and use your own `weka.jar` if you wish.

### Training with cross-validation

Configure your classifier in Weka, then right-click on the classifier options and choose "Copy configuration to clipboard." Your clipboard should contain something like this:

```
weka.classifiers.rules.OneR -B 6
```

Upload your ARFF file to delenn, then run Weka like so:

```
$ java -cp /opt/weka/weka-3.8.0.jar [paste classifier options] -t myfile.arff
```

E.g.,

```
$ java -cp /opt/weka/weka-3.8.0.jar weka.classifiers.rules.OneR -B 6 -t myfile.arff
```

### Training followed by prediction

In order to predict using a trained model, you need to save the model, and then load it to use as a predictor for the new instances.

First, train the model and save the result:

```
$ java -cp /opt/weka/weka-3.8.0.jar [paste classifier options] -d mymodel -t myfile.arff
```

The model will be saved to `mymodel` or whatever you specified after `-d`.

Next, to predict for new instances, create an ARFF file with the new instances (but missing the "class" column, i.e., the column you want to predict). This input file should match the format of the training file, i.e., all the attributes must be present (except the class). If you created the training file with StringToWordVector, refer to the [Weka for text processing](/notes/weka-text-processing.html) as extra care is needed to ensure the same attributes are present in the testing inputs.

CSV output:

```
$ java -cp /opt/weka/weka-3.8.0.jar [classifier] -l mymodel -T mynewdata.arff -classifications weka.classifiers.evaluation.output.prediction.CSV
```

Plaintext output:

```
$ java -cp /opt/weka/weka-3.8.0.jar [classifier] -l mymodel -T mynewdata.arff -classifications weka.classifiers.evaluation.output.prediction.PlainText
```

E.g.,

```
java -cp /opt/weka/weka-3.8.0.jar weka.classifiers.bayes.NaiveBayes -l mymodel -T iris-testing.arff -classifications weka.classifiers.evaluation.output.prediction.CSV
```

which gives,

```
inst#,actual,predicted,error,prediction
1,1:?,1:Iris-setosa,,1
2,1:?,2:Iris-versicolor,,1
3,1:?,1:Iris-setosa,,1
```

The interesting column is #3, predicted class. The number in front, e.g., `2:Iris-versicolor`, is the number for the nominal class attribute.