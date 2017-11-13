---
title: Deep learning
layout: note
---

# Deep learning

"Deep learning" is a buzzword like "big data" but nevertheless refers to a profoundly successful evolution of neural network architectures and training algorithms. The central idea is to take a multilayer perceptron, which typically has one hidden layer, and add several more layers. We then need an appropriate training algorithm that can handle the "vanishing gradient" problem and efficiently apply backpropagation to thousands or more weights at each layer. New activation functions, stochastic weight update, "dropout," and pooling are some of the techniques that make training many-layered neural networks possible.

By adding more layers, we allow the network to learn subtle properties of the data. We can even abandon careful feature engineering in many cases, and just let the various hidden layers learn their own complex representations. Convolutional neural networks are a prime example of this: some of the earliest layers apply various image manipulations (e.g., rotations, increased contrast, edge detection, etc.) in order to learn which features (perhaps diagonal high-contrast edges?) are best for the given data.

## Current landscape

Deep learning is currently all-the-rage. A recent *Nature* [paper](https://www.cs.toronto.edu/~hinton/absps/NatureDeepReview.pdf) summarizes the benefits of deep learning:

> Deep learning is making major advances in solving problems that have resisted the best attempts of the artificial intelligence community for many years. It has turned out to be very good at discovering intricate structures in high-dimensional data and is therefore applicable to many domains of science, business and government. In addition to beating records in image recognition and speech recognition, it has beaten other machine-learning techniques at predicting the activity of potential drug molecules, analysing particle accelerator data, reconstructing brain circuits, and predicting the effects of mutations in non-coding DNA on gene expression and disease. Perhaps more surprisingly, deep learning has produced extremely promising results for various tasks in natural language understanding, particularly topic classification, sentiment analysis, question answering and language translation.
>
> We think that deep learning will have many more successes in the near future because it requires very little engineering by hand, so it can easily take advantage of increases in the amount of available computation and data. New learning algorithms and architectures that are currently being developed for deep neural networks will only accelerate this progress.

Much of the engineering work for applying deep learning to a new problem is choosing the right network architecture (kinds of neurons, number of layers, numbers of neurons in each layer). The Asimov Institute's [Neural Network Zoo](http://www.asimovinstitute.org/neural-network-zoo/) is a good summary of the different common approaches. The figure below is a smaller version of their diagram. Click the figure to visit their page.

[![Neural Network Zoo](/images/neuralnetworkzoo.png)](http://www.asimovinstitute.org/neural-network-zoo/)

## Activation functions

The choice of activation function impacts speed of learning (in processing time), speed of learning (in epochs), and representational power (e.g., linear activation functions are a bad choice since multilayer networks are equivalent to single-layer linear networks). Deep learning research has introduced some new activation functions, but also continues to make use of some we've seen before.

### Sigmoid

![Sigmoid](/images/logistic-curve.png)

> We have already seen sigmoid units as output units, used to predict the probability that a binary variable is 1. Unlike piecewise linear units, sigmoidal units saturate across most of their domain — they saturate to a high value when $z$ (the input) is very positive, saturate to a low value when $z$ is very negative, and are only strongly sensitive to their input when $z$ is near 0. The widespread saturation of sigmoidal units can make gradient-based learning very diﬃcult. For this reason, their use as hidden units in feedforward networks is now discouraged. Their use as output units is compatible with the use of gradient-based learning when an appropriate cost function can undo the saturation of the sigmoid in the output layer. [Source](http://www.deeplearningbook.org/contents/mlp.html), page 195.

### Hyperbolic tangent (tanh)

![Hyperbolic tangent](/images/tanh.png)

> When a sigmoidal activation function must be used, the hyperbolic tangent activation function typically performs better than the logistic sigmoid. It resembles the identity function more closely, in the sense that $\tanh(0)=0$ while $\text{sigmoid}(0)=0.5$. Because tanh is similar to the identity function near 0, training a deep neural network resembles training a linear model so long as the activations of the network can be kept small. This makes training the tanh network easier. [Source](http://www.deeplearningbook.org/contents/mlp.html), page 195.

### Rectifier (relu)

![ReLU](/images/relu.png)

> Rectiﬁed linear units are easy to optimize because they are so similar to linear units. The only diﬀerence between a linear unit and a rectiﬁed linear unit is that a rectiﬁed linear unit outputs zero across half its domain. This makes the derivatives through a rectiﬁed linear unit remain large whenever the unit is active. The gradients are not only large but also consistent. The second derivative of the rectifying operation is 0 almost everywhere, and the derivative of the rectifying operation is 1 everywhere that the unit is active. This means that the gradient direction is far more useful for learning than it would be with activation functions that introduce second-order eﬀects. [Source](http://www.deeplearningbook.org/contents/mlp.html), page 193.

## Fancy layers

### Dropout

Keras docs: "Dropout consists in randomly setting a fraction of input units to 0 at each update during training time, which helps prevent overfitting."

## Keras

### Iris example

```python
from keras.models import Sequential
from keras.layers import Dense, Activation
from keras.utils.np_utils import to_categorical
import numpy

# load data
alldata = numpy.genfromtxt('iris.csv', delimiter=',', names=True,
                           dtype=(float, float, float, float, '|S15'))

# grab two things:
# - the unique class names, in a sorted list (setosa, versicolor, virginica)
# - every row's class replaced with a number (0,1,2)
class_uniq_ids, class_row_ids = numpy.unique(alldata['Class'], return_inverse=True)

# strip off the class column
alldata_noclass = numpy.array([list(row)[:-1] for row in alldata])
# now add on the numeric class (0,1,2 produced above) as a column
alldata_numeric = numpy.column_stack((alldata_noclass, class_row_ids))

# shuffle data
numpy.random.shuffle(alldata_numeric)

# figure out where to split
splitidx = int(len(alldata_numeric)*0.7)

# create train, test split
train, test = alldata_numeric[:splitidx,:], alldata_numeric[splitidx:,:]

# strip off class column
train_input = train[:,:-1]

# grab class column
train_labels = train[:,-1]
# turn class column into an int32 (to appease numpy)
train_labels = train_labels.astype(numpy.int32, copy=False)
# turn class column into one-hot encoding; this turns the one column into three columns
train_labels = to_categorical(train_labels, len(class_uniq_ids))

# do all the above for the test split
test_input = test[:,:-1]
test_labels = test[:,-1]
test_labels = test_labels.astype(numpy.int32, copy=False)
test_labels = to_categorical(test_labels, len(class_uniq_ids))

# build a neural net model
model = Sequential([
    Dense(10, input_dim=4),
    Activation('tanh'),
    Dense(len(class_uniq_ids)),
    Activation('tanh'),
    Activation('softmax')
    ])
model.compile(optimizer='sgd', loss='categorical_crossentropy', metrics=['accuracy'])

# train for 200 iterations
model.fit(train_input, train_labels, epochs=200)

# predict a single new (fake) iris with its four measurements
print(model.predict(numpy.array([[3.2, 4.5, 2.1, 4.3]])))

# test the model on the test split
# two numbers will print: loss (should be low) & accuracy (should be high)
print(model.evaluate(test_input, test_labels))
```

