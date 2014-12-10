---
layout: post
title: "Supervised Learning"
---

### [K-NN] (http://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm) Basic Approach
1. store all training examples
- For new(i.e. test) examples, find **k** nearest stored examples
* combine these **k** into the answer for the test example

How to choose a special k?:
Use tune set! [Cross-validation] (http://en.wikipedia.org/wiki/Cross-validation_(statistics))

#### For k-NN, good idea to normalize your data
* zero mean + unit standard deviation
* scale to [-1, 1] or [0, 1]

#### Some Variations
1. weigh neighbours by 1 / distance
* draw an N-dimension sphere and collect all points inside, then do a vote
    * curse of dimensionality: irrelevant features overwhelm informative features as dimension grows

### The standard (non-deep) structure of [Artificial neural network] (http://en.wikipedia.org/wiki/Artificial_neural_network)
* weights: persist between examples, though learning algorithm changes them slowly
* Different ANNs:
    * No Hidden Unit: perceptron
    * Several layers of Hidden Units: DEEP Neural Network
* Activation Functions (except for input units)
* forward propagation ("reasoning")
* train the threshold
* delta (learning) rule
* weight space
* early stopping
* epoch is one cycle through all training examples
* [cs540 Jim Gast@1999] (http://pages.cs.wisc.edu/~jgast/cs540/slides/19NeuralNets/)
* [cs540 Chuck Dyer@2014] (http://pages.cs.wisc.edu/~dyer/cs540/notes/nn.html)

#### Training/Learning the **-** (aka, bias)
Treating the bias as the n+1 feature (a special weight with input 1)

    # i = [1, N]
    #: THETA is bias
    If sum(Wi * OUTi) >= theta: # the step function
        return 1
    else:
        return 0

    # i = [1, N+1]
    # Where OUTn+1 = -1 and WEIGHTn+1 = theta
    If sum(Wi * OUTi) >= 0: # the step function
        return 1
    else:
        return 0

#### [Linear separability] (http://en.wikipedia.org/wiki/Linear_separability)
* If we can N features, can an N-1 dimensional hyperplane separate the **+** and **-** example
* Rewriting the "body" of the IF

        Line y = mx + b
        W1X1 + W2X2 = THETA is the "decision boundary"
        Solving for X2
            X2 = (THETA - W1X1) / W2 = (-W1/W2)X1 + THETA
* XOR is not linearly separable

##### [Perceptron convergence theorem] (http://annet.eeng.nuim.ie/intro/course/chpt2/convergence.shtml) by Rosenblatt
* If there is a set of weights that correctly classify the ( linearly seperable ) training patterns, then the learning algorithm will find one such weight set, w* in a finite number of iterations (Rosenblatt)
* if a set of examples is linearly separable then the delta rule will learn the concept (i.e. get all the training examples correct)

#### [backward propagation] (http://en.wikipedia.org/wiki/Backpropagation) ("learning")
* How to define the error or discrepency E of ANN?
    * use sqaured error as the [loss function] (http://en.wikipedia.org/wiki/Loss_function)
    * Error = 0.5 * (**y** - **Hw**) ** 2
        * The factor of 0.5 is included to cancel the exponent when differentiating. Later, the expression will be multiplied with an arbitrary **learning rate**, so that it does not matter if a constant coefficient is introduced now.
* How to learn from the calculated Error?
    1. Follow the direction of [Gradient descent] (http://en.wikipedia.org/wiki/Gradient_descent)
        * So the activiation function should be derivable
    - Finding the derivative of the error: CUSTOMIZED_ERROR
        * If in output layer, the CUSTOMIZED_ERROR could be defined as: Error * g'(self) (aka, the kth component of the error vector **y** - **Hw** * the gradient of this output component)
        * If in hidden layer, the CUSTOMIZED_ERROR would be affected by many succeed nodes, so CUSTOMIZED_ERROR = g'(self) * sum(succeed_weight * succeed_delta)
        * If there is an error, the more a previous layer node contributes the error (weight) the more punishments the previous input should undertake
        * The punishments would propagate until the input layer
    - update weight (the gradient) by: OUTj * CUSTOMIZED_ERRORk
* [Neural network tutorial video] (https://www.youtube.com/watch?v=zpykfC4VnpM)
* [An Example of Back Propagation Algorithm] (https://www4.rgu.ac.uk/files/chapter3%20-%20bp.pdf)

1. Error = 0.5 * sum[(TEACHi - Outi) ** 2]
- Error = 0.5 * sum[(TEACHi - f(sum(Wi,j * OUTj))) ** 2]
- Error = 0.5 * sum[(TEACHi - f(sum(Wi,j * f(sum(Wj,k * OUTk))))) ** 2]

##### [Delta rule] (http://en.wikipedia.org/wiki/Delta_rule)
In machine learning, the delta rule is a gradient descent learning rule for updating the weights of the inputs to artificial neurons in single-layer neural network.

In single-layer network, its Delta Rule is the derivative of the error

Can improve by:

1. decreasing threshold
- if OUTj < 0: then lessen reduce the weight on this "negative" evidence

#### Weight Space (one point in weight space labels **all** points in feature space)
* the space of all possible values of all of the neuron's weights
* the *gradient descent* is performed in weight space

#### Four Views of Hidden Units (not disjoint)
1. perceptron
- Learn good derived features (e.g. edge detection in computer vision)
- Divide Feature Space int O(2 ** N) Regions
- Learn a set of good basis functions for learning complex functions

#### Drop Out (Network)
The idea is simple: During the training process, hidden nodes and their connections are randomly dropped from the neural network.

It adresses the main problem in machine learning, that is **overfitting**. It does so by “dropping out” some unit activations in a given layer, that is setting them to zero. Thus it prevents co-adaptation of units and can also be seen as a method of ensembling many networks sharing the same weights.

The idea has a biological inspiration. When a child is conceived, it receives half its genes from each parent. Because of this the genes cannot rely on a presence of any other particular genes and it forces them to be useful on their own and play well with **unknown others**.

1. During each forward propagation & back propagation, for each input example, each Hidden Unit has an independent probability **P**(e.g. 0.5) to set its activation to zero (which means dropout)
- During testing, multiply each weight by (1 - P) since that is its expected value during training

Viewing as an Ensemble. All of these reped in our full ANN, with one selected during drop out

* [Neural Network Dropout Training] (http://visualstudiomagazine.com/articles/2014/05/01/neural-network-dropout-training.aspx)
* [Regularizing neural networks with dropout and with DropConnect] (http://fastml.com/regularizing-neural-networks-with-dropout-and-with-dropconnect/)

#### ANN wrap up
* lots of current excitement & success
* simple idea, gradient descent
    * limitation of gradient descent is local minima
* computationally demanding
* hard to understand what was learned ("rule extraction")


### [Support Vector Machines] (http://en.wikipedia.org/wiki/Support_vector_machine)
Three Key Ideas

1. Pick the *best* separating line
    * largest margin
- Pay penalty for misclassified training examples
    * min COST = 0.5 ||**w**|| ** 2 + 0.5 lambda sum[(TEACHERi - PREDICTEDi) ** 2]
        * **w** want big margin
        * lambda: a weighting term (use tune set to choose good value)
        * In fact, min COST = model_complexity + error_rate
- Switch (via "similarity" functions, a.k.a. [kernels] (http://en.wikipedia.org/wiki/Kernel_method)) to a different representation, where percei suffice

We will find a good setting for the weights using gradient descent (instead of linear|[quadratic programming] (http://en.wikipedia.org/wiki/Quadratic_programming))

#### Third SVM Idea - kernels
* create new features NEW_FEATUREi = similarity(NEW_EXAMPLE, TRAINING_EXAMPLEi) of new example
* then run SVM in this new feature set
* Note this is also Hidden Units did

A simple, but compelling example of the Power of Kernels

##### Reference
* [The Art of Programming] (https://github.com/luozhaoyu/The-Art-Of-Programming-By-July/blob/master/ebook/zh/07.02.svm.md)
