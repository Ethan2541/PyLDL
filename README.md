# PyLDL

Python Lightweight Deep Learning is a project that aims to create a dedicated library to implement basic neural networks in Python. It is an assignement part of the course [Machine Learning](https://dac.lip6.fr/master/ml/) from the first year of Sorbonne Université's computer science master.


## Linear Modules

Linear modules are the most basic elements of any neural network. They are thoroughly tested in the notebook `experiments/linear_network.ipynb`.

### Linear Module

The `Linear` module implements a basic linear layer, whose forward pass is a dot product with its parameters (or weights) and the input. It is one of the most fundamental component of any neural network in `PyLDL`. It is usually followed by an activation module, and often encapsulated in `Sequential`

### Mean Square Error loss function

The `MSELoss` simply implements the mean square error loss between the predicted and actual outputs. It is the mean of the squared euclidean norm of the difference of both outputs along the features axis. Instead of returning a vector of these norms, we chose to only return the mean as it is more convenient to plot the loss across the different epochs of the training phase.

The formula of the `MSELoss` and its derivative are given by the following equations:
              $$Loss(y, \hat{y}) = \frac{1}{N} \sum_{i=1}^{N} \left\| y_i - \hat{y}_i \right\|_2^2$$
              $$\frac{\partial \text{Loss}}{\partial \hat{y}} = -2 (y - \hat{y})$$



### Binary Cross Entropy loss function

The `BCELoss` implements the binary cross-entropy loss between the predicted and actual outputs. This loss is decent when handling image reconstruction tasks. Compared to the MSE loss, whose outputs are smoothed, the outputs of the BCE loss are more extreme and either close to 0 or 1. In the case of white and black digit images, it makes the white digits sharper, contrasting with the black background.

Its formula is:
$$Loss(y, \hat{y}) = -\frac{1}{N} \sum_{i=1}^{N} \left[ y_i \log(\hat{y}_i + \epsilon) + (1 - y_i) \log(1 - \hat{y}_i + \epsilon) \right]$$  
where $\epsilon$ is a small value to avoid logarithm of zero. The derivative of the binary cross-entropy is given by:                
$$\frac{\partial \text{Loss}}{\partial \hat{y}} = \frac{\hat{y} - y}{(\hat{y} + \epsilon)(1 - \hat{y} + \epsilon) N}$$

### Cross Entropy loss function

The `CrossEntropyLoss` implements the cross-entropy loss between the predicted and actual outputs. It first applies the softmax function to the predicted outputs to obtain the probability distribution over the classes, ensuring numerical stability by subtracting the maximum predicted value. The loss is then calculated as the mean of the negative log likelihood of the true class probabilities. By averaging over all samples, the returned value provides a single scalar loss, which is more convenient to plot across different epochs during the training phase. 

The formula of the `CELoss` and its derivative are given by:
$$Loss(y, \hat{y}) = \frac{1}{N} \sum_{i=1}^{N} \left(-\sum_{c=1}^{C} y_{i,c} \hat{y}_ {i,c} + \log\sum_{c=1}^{C} \exp(\hat{y}_ {i,c}) \right)$$
$$\frac{\partial \text{Loss}}{\partial \hat{y}} = \frac{\exp(\hat{y}_ {i,c} - \max(\hat{y}_ {i}))}{\sum_c \exp(\hat{y}_ {i,c} - \max(\hat{y}_ {i}))} - y_{i,c}$$

## Nonlinear Modules

Nonlinear modules are basically activation functions which inherit the characteristics of the module `Activation`, available in `pyldl/activations.py`. As such, they only have a forward and a backward pass. They don't have any parameter to update or to reset. Tests can be found in `experiments/activation_network.ipynb`

### Hyperbolic Tangent

First, we considered the hyperbolic tangent:
$$tanh(x) = \frac{e^x - e^{-x}}{e^x + e^{-x}}$$

It is easily implemented thanks to `numpy`'s built-in function `tanh`. Its derivative is given by:
$$tanh'(x) = 1 - tanh^2(x)$$

### Sigmoid

We also implemented the sigmoid function:
$$\sigma(x) = \frac{1}{1 + e^{-x}}$$

Its derivative is:
$$\sigma'(x) = \sigma(x)(1 - \sigma(x))$$

### Softmax

In order to represent distributions in the case of multiclass for example, we implemented a `Softmax` module. The forward pass is given by:
$$Softmax_i(x) = \frac{e^{x_i}}{\sum_k e^{x_k}}$$

The derivative of the softmax activation function is:
$$Softmax_i'(x) = Softmax_i(x) * (1 - Softmax_i(x))$$

### Rectified Linear Unit

The ReLU activation function is very useful, especially for convolutional networks. Its expression is simple:
$$ReLU(x) = \max(0,x)$$

Its derivative is thus either $0$ (if x is negative or zero) or $1$.


## Encapsulation

This section focuses on the creation and the training of a network. Experiments can be found in `experiments/sequential_network.ipynb`. The tasks and the random seed are the same as the two previous notebooks. As such, they should yield the exact same results.

### Sequential Network

The class `Sequential`, available in `pyldl/layers.py`, represents a network and encapsulates a list of `Module` objects. Its forward pass is the sequence of forward passes, each having as input the output of the previous layer. The backward pass is designed in the same fashion. Additionnally, the backpropagation is ensured by both the `backward()` and the `update_parameters()` methods.

### Optimization

In the file `pyldl/optimizers.py`, we implemented a class `Optim` which encompasses a whole step of a network (forward pass + backpropagation + parameters update) and a function `SGD` which trains the network over a definite amount of epochs. The training phase of `SGD` is a minibatch gradient descent (depending on the size of the batches, the descent can either be stochastic or batch).

### A specific type of network: the AutoEncoder

An autoencoder is a network divided into two parts: the encoder and the decoder. Basically, the encoder gives a representation of the input in a latent space whose dimension is smaller than the original space. Then, the decoder returns a new representation (that should be faithful to the given input) in the original space. Also, both the decoder and the encoder should have the same amount of layers, set in a sort of symetric way. For example, if the encoder encapsulates: [Linear(64,16), Linear(16,2)], the decoder should have the same layers in reverse order, with inverted in-features / out-features: [Linear(2,16), Linear(16,64)]. The class `AutoEncoder` is available in `pyldl/layers.py`.

## Convolutional Network
The use of convolutional neural networks offers numerous advantages for image processing. Convolution allows capturing local patterns, reducing the number of learnable parameters, achieving translation invariance, and reducing data dimensionality. This makes it particularly suitable for images, temporal sequences, or signals.

### Conv1D
The module `Conv1D(k_size, chan_in, chan_out, stride)` contains a parameter matrix having the size `(k_size, chan_in, chan_out)`, which corresponds to chan_out filters of size `(k_size, chan_in)`. Its forward method takes as input a batch of size `(batch, length, chan_in)` and outputs a matrix of size `(batch, (length - k_size)/stride + 1, chan_out)`.

### MaxPool1D
The `MaxPool1D` module, devoid of learnable parameters, operates with a kernel size `k_size` and a stride length `stride`. Its forward method accepts input batches of size `(batch, length, chan_in)` and yields an output matrix of dimensions `(batch, (length - k_size)/stride + 1, chan_in)`.

### Flatten
The `Flatten` module transforms input batches of size `(batch, length, chan_in)` into a matrix of dimensions `(batch, length \times chan_in)`.
