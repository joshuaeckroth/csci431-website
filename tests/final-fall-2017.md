---
layout: note
title: Final exam for Fall 2017
categories: [tests]
---

# Final exam for Fall 2017

Submit your solutions on bitbucket in a repository named csci431-final or similar. Create a folder for each question. **Answers are due by Thur Dec 14, 11:59pm.** You may only look at these course notes, any linked page, and standard documentation for Keras and scikit-learn. The exam is out of 100 points (but weighted 20% of your overall grade).

## Task 1 (10pts)

Describe a scenario where k-fold cross validation would produce an accurate estimate of the performance of a model while a simple train/test split would not.

## Task 2 (20pts)

Grab the recent [Orlando Crime Data](https://data.cityoforlando.net/Orlando-Police/OPD-Crimes/4y9m-jbmz) (click Export for CSV links). Build a naïve Bayes model for predicting the Case Offense Category from Case Offense Location Type and time of day (hour). Consider using scikit-learn's [MultinomialNB](http://scikit-learn.org/stable/modules/generated/sklearn.naive_bayes.MultinomialNB.html#sklearn.naive_bayes.MultinomialNB) class. Demonstrate using the model to predict a new case.

## Task 3 (10pts)

Suppose we have the following network, where ROWS=COLS=256 and CHANNELS=3.

```
model = Sequential()
model.add(Conv2D(64, (3,3), input_shape=(ROWS, COLS, CHANNELS)))
model.add(Activation('relu'))
model.add(Conv2D(64, (3,3)))
model.add(Activation('relu'))
model.add(MaxPooling2D(pool_size=(4,4)))
model.add(Conv2D(64, (3,3)))
model.add(Activation('relu'))
model.add(Conv2D(64, (3,3)))
model.add(Activation('relu'))
model.add(MaxPooling2D(pool_size=(4,4)))
model.add(Flatten())
model.add(Dropout(0.5))
model.add(Dense(400))
model.add(Activation('relu'))
model.add(Dropout(0.5))
model.add(Dense(200))
model.add(Activation('softmax'))
```

Explain every number (where the number is specified by the user or how the number is calculated) in the network summary below. You may want to review the notes on delenn: `/home/jeckroth/csci431-public/cnn/model-dimensions.py`.

```
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
conv2d_19 (Conv2D)           (None, 254, 254, 64)      1792      
_________________________________________________________________
activation_28 (Activation)   (None, 254, 254, 64)      0         
_________________________________________________________________
conv2d_20 (Conv2D)           (None, 252, 252, 64)      36928     
_________________________________________________________________
activation_29 (Activation)   (None, 252, 252, 64)      0         
_________________________________________________________________
max_pooling2d_10 (MaxPooling (None, 63, 63, 64)        0         
_________________________________________________________________
conv2d_21 (Conv2D)           (None, 61, 61, 64)        36928     
_________________________________________________________________
activation_30 (Activation)   (None, 61, 61, 64)        0         
_________________________________________________________________
conv2d_22 (Conv2D)           (None, 59, 59, 64)        36928     
_________________________________________________________________
activation_31 (Activation)   (None, 59, 59, 64)        0         
_________________________________________________________________
max_pooling2d_11 (MaxPooling (None, 14, 14, 64)        0         
_________________________________________________________________
flatten_5 (Flatten)          (None, 12544)             0         
_________________________________________________________________
dropout_8 (Dropout)          (None, 12544)             0         
_________________________________________________________________
dense_9 (Dense)              (None, 400)               5018000   
_________________________________________________________________
activation_32 (Activation)   (None, 400)               0         
_________________________________________________________________
dropout_9 (Dropout)          (None, 400)               0         
_________________________________________________________________
dense_10 (Dense)             (None, 200)               80200     
_________________________________________________________________
activation_33 (Activation)   (None, 200)               0         
=================================================================
Total params: 5,210,776
Trainable params: 5,210,776
Non-trainable params: 0
_________________________________________________________________
```

## Task 4 (30pts)

Starting with the code given below, create a music genre classifier using a Keras neural network. Each song has 25000 data points. Training and testing data have already been split. You must achieve an accuracy of at least 52%.

```
import librosa
import librosa.feature
import librosa.display
import glob
import numpy as np
#import matplotlib.pyplot as plt
from keras.models import Sequential
from keras.layers import Dense, Activation
from keras.utils.np_utils import to_categorical

def extract_features_song(f):
    y, _ = librosa.load(f)

    # get Mel-frequency cepstral coefficients
    mfcc = librosa.feature.mfcc(y)
    # normalize values between -1,1 (divide by max)
    mfcc /= np.amax(np.absolute(mfcc))

    return np.ndarray.flatten(mfcc)[:25000]

def generate_features_and_labels():
    all_features = []
    all_labels = []

    genres = ['blues', 'classical', 'country', 'disco', 'hiphop', 'jazz', 'metal', 'pop', 'reggae', 'rock']
    for genre in genres:
        sound_files = glob.glob('/bigdata/data/marsyas/genres/'+genre+'/*.au')
        print('Processing %d songs in %s genre...' % (len(sound_files), genre))
        for f in sound_files:
            features = extract_features_song(f)
            all_features.append(features)
            all_labels.append(genre)

    # convert labels to one-hot encoding
    label_uniq_ids, label_row_ids = np.unique(all_labels, return_inverse=True)
    label_row_ids = label_row_ids.astype(np.int32, copy=False)
    onehot_labels = to_categorical(label_row_ids, len(label_uniq_ids))
    return np.stack(all_features), onehot_labels

features, labels = generate_features_and_labels()

print(np.shape(features))
print(np.shape(labels))

training_split = 0.8

# last column has genre, turn it into unique ids
alldata = np.column_stack((features, labels))

np.random.shuffle(alldata)
splitidx = int(len(alldata) * training_split)
train, test = alldata[:splitidx,:], alldata[splitidx:,:]

print(np.shape(train))
print(np.shape(test))

train_input = train[:,:-10]
train_labels = train[:,-10:]

test_input = test[:,:-10]
test_labels = test[:,-10:]

print(np.shape(train_input))
print(np.shape(train_labels))

model = # TODO

# TODO (model.compile, model.fit, etc.)

loss, acc = model.evaluate(test_input, test_labels, batch_size=32)

print("Loss: %.4f, accuracy: %.4f" % (loss, acc))
```

## Task 5 (30pts)

Starting with the code given below, create a classifier for handwritten mathematical symbols (images sized 30x30). Training and testing data have already been split. You must achieve an accuracy of at least 77%.

```
import csv
from PIL import Image as pil_image
import keras.preprocessing.image

# load all images (as numpy arrays) and save their classes

imgs = []
classes = []
with open('/bigdata/data/HASYv2/hasy-data-labels.csv') as csvfile:
    csvreader = csv.reader(csvfile)
    i = 0
    for row in csvreader:
        if i > 0:
            img = keras.preprocessing.image.img_to_array(pil_image.open("/bigdata/data/HASYv2/" + row[0]))
            # neuron activation functions behave best when input values are between 0.0 and 1.0 (or -1.0 and 1.0),
            # so we rescale each pixel value to be in the range 0.0 to 1.0 instead of 0-255
            img /= 255.0
            imgs.append((row[0], row[2], img))
            classes.append(row[2])
        i += 1

# shuffle the data, split into 80% train, 20% test

import random
random.shuffle(imgs)
split_idx = int(0.8*len(imgs))
train = imgs[:split_idx]
test = imgs[split_idx:]

import numpy as np

train_input = np.asarray(list(map(lambda row: row[2], train)))
test_input = np.asarray(list(map(lambda row: row[2], test)))

train_output = np.asarray(list(map(lambda row: row[1], train)))
test_output = np.asarray(list(map(lambda row: row[1], test)))

from sklearn.preprocessing import LabelEncoder
from sklearn.preprocessing import OneHotEncoder

# convert class names into one-hot encoding

# first, convert class names into integers
label_encoder = LabelEncoder()
integer_encoded = label_encoder.fit_transform(classes)

# then convert integers into one-hot encoding
onehot_encoder = OneHotEncoder(sparse=False)
integer_encoded = integer_encoded.reshape(len(integer_encoded), 1)
onehot_encoder.fit(integer_encoded)

# convert train and test output to one-hot
train_output_int = label_encoder.transform(train_output)
train_output = onehot_encoder.transform(train_output_int.reshape(len(train_output_int), 1))
test_output_int = label_encoder.transform(test_output)
test_output = onehot_encoder.transform(test_output_int.reshape(len(test_output_int), 1))

num_classes = len(label_encoder.classes_)
print("Number of classes: %d" % num_classes)


model = # TODO

# model.compile, model.fit, etc.

score = model.evaluate(test_input, test_output, verbose=2)
print('Test loss:', score[0])
print('Test accuracy:', score[1])
```


