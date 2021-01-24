# neural networks

classification:
- a **training set** of **instances**
- each instance has **attributes**
- each attribute has a **value**
- usually the last attribute is the **class**
- ...classify 'em

regression:
- it represents the approximation of a function
- any kind of function, not just $f:\R^n \rightarrow \R$
- only the output is continuous, some inputs may be discrete or non-numeric

classification and regression have the same basic idea - learn the relationship between input $x$ and output $y$
- the only difference is the type of output
  - discrete - the class, for classification
  - continuous - the function $h$ s.t. $h(x) \approx y(x)$
- for each training instance, the desired output is given - **supervised training**
- the neural networks in this course are generally used for classification and regression

there was some stuff here about real neurons and fake neurons and how they're alike but that doesnt seem too important to me

## the perceptron
a simple neuron
- $n$ inputs, $x_i$
- $n$ weights, $w_i$
- one output, $y$
- a **bias**, $\theta$
- the neuron fires if the weighted sum of the inputs exceeds the bias
$$y = F \left(\sum_{i=1}^n{w_ix_i - \theta}\right)$$
- the activation function $F(X)$ can be:
  - sign ($+1, X\ge0; -1, X<0$)
  - step ($1, X\ge0; 0, X<0$)
  - other stuff? later

the $n$-dimensional input space is divided by a hyperplane:
    $$\sum_{i=1}^n{w_ix_i - \theta = 0}$$

we can simplify things by treating the bias as an extra (input, weight) pair: $w_{n+1}x_{n+1}$, where $w_{n+1} = \theta$ and $x_{n+1} = -1$
$$y = F \left(\sum_{i=1}^{n+1}{w_ix_i}\right)$$

### learning
learning takes place by successfully adjusting weights (including the bias) to reduce the difference between the actual and desired outputs for all training data (the *error*)

- the error is $e = y_d - y$, where $y_d$ is the desired output
- if the error is positive, we need to increase $y$
- if it's negative, we need to decrease it
- change the weights by $\Delta w = \alpha \cdot x \cdot e$, where $\alpha$ is the **learning rate**

the perceptron is limited - it can only represent *linearly separable* functions

### adaline
an evolution of the perceptron

- linear activation function
$$ y = \sum_{i=1}^{n+1}{w_ix_i}$$
- mean squared error
$$E_i = \frac{1}{2}(y_{di} - y{i})^2$$
- the error for the entire training set:
$$E = \frac{1}{n}\sum_{i=1}^{n}{E_i}$$
- the delta rule ($f$ is the derivable activation function):
$$\Delta_iw_j = \alpha \cdot x_{ij} \cdot f'(y_i) \cdot e_i$$
    - the linear activation function derives to $1$: $f'(y_i)$ = 1
    - therefore, ultimately this is the same weight change as the perceptron: $\Delta w_j = \alpha x_{ij} e_i$

## the multilayer perceptron
if you slap multiple perceptrons together you can make better stuff

the multilayer perceptron is a *feed-forward neural network* with one or more **hidden layers**:
- one input layer (does nothing, just a few inputs)
- one or more hidden layers
- one output layer

the input signals are propagated through the layers of the network

### activation functions
- the **unipolar sigmoid**:
$$f(x) = \frac{1}{1+e^{-x}}$$
- the **bipolar sigmoid**
$$f(x) = \frac{1-e^{-2x}}{1+e^{-2x}}$$
- a single-layer perceptron is not helped by a nonlinear activation function
- a multilayer perceptron with linear activation functions is no better than a single-layer one
- therefore, you need a multilayer perceptron with a nonlinear activation function

### learning
- similar to a perceptron
- the network receives the input vectors and computes the output vectors
- if there is an error, the weights are adjusted

for learning, we use the **backpropagation algorithm**
it has two phases:
- the network receives the input and propagates the signal *forward* , layer by layer, to generate the output
- the *error* signal is propagated *backward*, adjusting the weights of the network

the method is as such; mentally add $\cdot(p)$ or $\cdot(p+1)$ to everything since they all depend on the current input vector and update for the next one

#### initialization
- set the weights (incl. biases) of the networks to small, non-zero random values
  - the $[-0.1, 0.1]$ interval works well
  - an heuristic recommends values in $\left(\frac{-2.4}{F_i}, \frac{2.4}{F_i}\right)$, where $F_i$ is the number of inputs for the neuron
- the initialization is done separately for each neuron
- next, we will use the indices $i = \overline{1, n}$ for the input layer, $j = \overline{1, m}$ for the hidden layer, $k = \overline{1, l}$ for the output layer
#### activation
- hidden layer outputs:
$$y_j(p) = sigmoid\left(\sum_{i=1}^{n+1}{w_{ij}x_{i}}\right)$$
- output layer outputs:
$$y_k(p) = sigmoid\left(\sum_{j=1}^{m+1}{w_{jk}y_{j}}\right)$$

#### backpropagation
for the output layer:
- calculate the error: $e_k = y_{dk} - y_{k}$
- calculate the **error gradient** for the neurons in the output layer, with the delta rule
$$\delta_k = f'(y_k) \cdot e_k$$
    - for the unipolar sigmoid, $f'(y_k) = f(y_k)(1 - f(y_k))$
    - for the bipolar sigmoid, $f'(y_k) = \frac{a}{2}(1-f(y_k))(1+f(y_k))$
- weight corrections:
$$
\Delta w_{jk} = \alpha \cdot y_j \cdot \delta_k
$$
- update the weights:
$$
w_{jk} \leftarrow w_{jk} + \Delta w_{jk}
$$
   
for the hidden layer:
- error gradient:
$$\delta_j = f'(y_j) \cdot \sum_{k=1}^{l}{\delta_k w_{jk}}$$
- weight corrections:
$$\Delta w_{ij} = \alpha \cdot x_i \cdot \delta_j$$
- update the weights:
$$w_{ij} \leftarrow w_{ij} + \Delta w_{ij}$$

if there are other hidden layers, proceed similarly

#### iteration
- go through the other inputs as well
- once all the inputs are done, an **epoch** is finished
- repeat until mean square error for the epoch reaches an acceptable threshold or until the number of epochs is reached

### types of training

- incremental learning (online learning)
  - the weights are updated after each training vector (like above)
- batch learning
  - only the weight corrections update after each training vector
  - the weights are updated only once at the end of the epoch
  - advantage: the results no longer depend on the order of the training vectors

### methods to accelerate learning
- sometimes, it helps to use the bipolar sigmoid instead of the unipolar sigmoid
- *momentum*:
  - the generalized delta rule: 
    $$\Delta w_{jk}(p) = \beta \cdot \Delta w_{jk}(p-1) + \alpha \cdot y_j(pp) \cdot \delta_k(p)$$
  - the difference is that the previous weight correction is multiplied by $\beta$, the **momentum constant**
  - usually $\beta$ is around 0.95
- adaptive learning rate:
  two heuristics can be applied to accelerate convergence and prevent instability
  - if $\Delta E$ has the same sign for a few epochs, consider increasing $\alpha$
  - if $\Delta E$ alternates in sign for a few epochs, consider decreasing $\alpha$
    
  to consolidate:
  - let $r$ = ratio, $d$ = *down*, $u$ = *up*
  - let $E_p$ the sum of quadratic errors in the current epoch
  - if $E_p > r * E_{p-1}$, $\alpha \leftarrow \alpha \cdot d$
  - if $E_p < E_{p-1}$, $\alpha \leftarrow \alpha \cdot u$
  - typical values: $r = 1.04$, $d = 0.7$, $u = 1.05$
- in order to not saturate the sigmoid functions, inputs and outputs are usually scaled so they don't quite reach the ends of their intervals - $[0.1, 0.9]$ or $[0.9, 0.9]$ depending on the kind of sigmoid used
- if the network is used for regression and not classification, the activation function of the output neurons can be linear (0 to 1) or semi-linear (-1 to 1) instead of sigmoid
- establishing the right number of layers and neurons per layer can be tricky
  - heuristic: Kudrycki's rule
  - for a single hidden layer, three times as many hidden layer neurons as outputs: $m = 3 * l$
  - for 2 hidden layers: $m_1 = 3 * m_2$
  
## deep networks
- more layers
- simpler activation functions (ReLU)
- cost functions based on the MLE instead of the MSE
- training algorithms: SGD, RMSProp, Adam, etc
  - instead of Backpropagation, RProp, Levenberg-Marquardt, etc
- other methods of weight initialization, regularization, pre-training, etc

### unstable gradients
- if you have a lot of layers, the first few layers won't train too well
- you need different training methods

### ReLU activation function
- rectified linear unit
$f(x) = max(0, x)$
- leaky ReLU
$f(x) = x $ if $x>0$ else $0.01x$
- parametric ReLU (PReLU)
$f(x) = x $ if $x > 0$ else $ax$, $a$ can be learned

### stochastic gradient descent
- for current problems, you can have a *lot* of training instances
- pick a random subset each epoch instead of all of them
  
## conclusions
- perceptron can do linearly separable functions
- multilayer perceptron can approximate anything
- backpropagation is the most used training algorithm for multilayer perceptrons
- deep networks have a lot of layers and a lot of problems and a lot of ways to solve those problems
