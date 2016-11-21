---
title: Weka for text processing
layout: note
---

# Weka for text processing

Download Weka from the [Weka website](http://www.cs.waikato.ac.nz/ml/weka/downloading.html), and download the "stable" version for your machine.

## Overview

As discussed in the [text processing](/notes/text-processing.html) notes, text files typically need to be converted into "feature vectors" before machine learning algorithms can be applied. The most common feature vector is a vector where each dimension is a different word, and the value in that dimension is either 0/1 binary value (yes or no the word is in that document), an integer count (the frequency of the word in that document), or a real value (often the tf-idf score of that word in that document).

The ARFF files we will be working with will always look like this:

```
@relation some-description-of-the-data
@attribute contents string
@attribute class {ham,spam}

@data
'Go until jurong point, crazy.. Available only in bugis n great world la e buffet... Cine there got amore wat...',ham
'Ok lar... Joking wif u oni...',ham
'Free entry in 2 a wkly comp to win FA Cup final tkts 21st May 2005. Text FA to 87121 to receive entry question(std txt rate)T&Cs apply 08452810075over18s',spam
'U dun say so early hor... U c already then say...',ham
'Nah I dont think he goes to usf, he lives around here though',ham
...
```

Of course, the classes (ham/spam) may change. These files are very easy to generate, should you actually wish to use text classification for your own purposes.

We'll practice with the [sms-spam.arff](/sms-spam.arff) file, which has examples of SMS text msg ham and spam. This collection was produced by Tiago A. Almeida from the Federal University of Sao Carlos and José María Gómez Hidalgo from the R&D Department of Optenet. You may find [their paper](http://dl.acm.org/citation.cfm?id=2034742) interesting; it examines the performance of various machine learning algorithms on this data set.

## Steps

### Strings to binary feature vectors

Load the ARFF file and choose a filter.

![Weka](/images/weka-preprocess-1.png)

Choose the StringToWordVector filter, in the Unsupervised > Attribute section.

![Weka](/images/weka-preprocess-2.png)

Click the filter name to get more options.

![Weka](/images/weka-preprocess-3.png)

Change "attributeIndices" to "1" to just select the first attribute (the contents of the SMS message). Change "attributeNamePrefix" to "word-" so that all the new features look like "word-foo" and "word-bar", etc. Change "wordsToKeep" to "100000" to keep them all.

![Weka](/images/weka-preprocess-4.png)

Click "Apply"

![Weka](/images/weka-preprocess-5.png)

Now you'll have lots more attributes. You can click "Undo" if you don't like it.

![Weka](/images/weka-preprocess-6.png)

Everything works better if the class (ham/spam) is last. After our filter, it's not anymore. Let's fix that. Choose a new filter, "Reorder," under Unsupervised > Attribute. Click the filter name to change its properties. Type "2-last,1" to make the new order the second-to-last attributes followed by the first attribute. This puts the first attribute at the end.

![Weka](/images/weka-preprocess-7.png)

Click "Apply." Now the class (ham/spam) should be the last attribute again.

![Weka](/images/weka-preprocess-8.png)

### Strings to frequency vectors

Follow the steps above ("Strings to binary feature vectors") except use the following changes in the StringToWordVector filter:

Change "outputWordCounts" to "True."

![Weka](/images/weka-preprocess-9.png)

### Strings to tf-idf vectors

Follow the steps above ("Strings to binary feature vectors") except use the following changes in the StringToWordVector filter:

- Change "IDFTransform" to "True"
- Change "TFTransform" to "True"
- Change "normalizeDocLength" to "Normalize all data"

![Weka](/images/weka-preprocess-10.png)

## Applying the same transformations for prediction

Words in a new set of documents may not exactly match those in the training set. But the training set only contains attributes for words found in the training set (and possibly not all such words if the "wordsToKeep" parameter was small). However, in order to use a trained model for prediction, all of the same attributes must be present in the test ARFF file.

To ensure the training ARFF and testing ARFF have the same attributes, we need to apply the same transformations to each dataset. Thus, we need to write the transformations in a script, to be run from the command line. Furthermore, when applying StringToWordVector on the test dataset, we need to give the StringToWordVector filter the training ARFF file.

Here are all the steps shown above for TF-IDF feature vectors as a series of command-line statements, in two scripts. First, `train.sh`:

```shell
#!/bin/sh

java -cp /opt/weka/weka-3.8.0.jar weka.filters.unsupervised.attribute.StringToWordVector \
        -R 1 -P word- -W 100000 -prune-rate -1.0 -T -N 1 \
        -stemmer weka.core.stemmers.NullStemmer -stopwords-handler weka.core.stopwords.Null \
        -M 1 -tokenizer "weka.core.tokenizers.WordTokenizer -delimiters \" \\\\r\\\\n\\\\t.,;:\\\\\'()\!?\\\\\\\"\"" \
        -i sms-spam.arff -o sms-spam-1.arff

java -cp /opt/weka/weka-3.8.0.jar weka.filters.unsupervised.attribute.Reorder \
        -R 2-last,first -i sms-spam-1.arff -o sms-spam-2.arff

java -cp /opt/weka/weka-3.8.0.jar weka.classifiers.bayes.NaiveBayes \
        -t sms-spam-2.arff -d mymodel
```

Next, make a test ARFF file:

```
@relation sms-spam-test
@attribute contents string
@attribute class {ham,spam}

@data
'This is a test of the spam detector. Is this message spam?',?
'Buy now! Earn money! Cash prize!',?
'Hey honey, can you pick up peanut butter',?
```

And run `test.sh`:

```shell
#!/bin/sh

# notice use of batch option: -b
# notice input of training data: -i sms-spam.arff, plus testing input/output: -r and -s
# processed training data must be saved again, for some reason: -o sms-spam-1.arff
# (otherwise it dumps the result to the console)

java -cp /opt/weka/weka-3.8.0.jar weka.filters.unsupervised.attribute.StringToWordVector \
        -b -R 1 -P word- -W 100000 -prune-rate -1.0 -T -N 1 \
        -stemmer weka.core.stemmers.NullStemmer -stopwords-handler weka.core.stopwords.Null \
        -M 1 -tokenizer "weka.core.tokenizers.WordTokenizer -delimiters \" \\\\r\\\\n\\\\t.,;:\\\\\'()\!?\\\\\\\"\"" \
        -i sms-spam.arff -o sms-spam-1.arff -r sms-spam-test.arff -s sms-spam-test-1.arff

java -cp /opt/weka/weka-3.8.0.jar weka.filters.unsupervised.attribute.Reorder \
        -R 2-last,first -i sms-spam-test-1.arff -o sms-spam-test-2.arff

java -cp /opt/weka/weka-3.8.0.jar weka.classifiers.bayes.NaiveBayes \
        -T sms-spam-test-2.arff -l mymodel -classifications weka.classifiers.evaluation.output.prediction.CSV
```

Result:

```
=== Predictions on test data ===

inst#,actual,predicted,error,prediction
1,1:?,2:spam,,1
2,1:?,2:spam,,1
3,1:?,1:ham,,1
```

