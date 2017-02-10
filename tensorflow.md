# Graph and Session
* A graph defines the computation. It doesn’t compute anything, it doesn’t hold any values, it just defines the operations that you specified in your code.
* A session allows to execute graphs or part of graphs. It allocates resources (on one or more machines) for that and holds the actual values of intermediate results and variables. One can create a session with tf.Session, and be sure to use a context manager, or tf.Session.close(), because all ressources of the session are saved. To run some graph element, you should use the function .run(graph_element, feed_dic), it return values, or list of values, if a list of graph elements was passed.

# Variable
## Some parameters
* Setting trainable=False keeps the variable out of the GraphKeys.TRAINABLE_VARIABLES collection in the graph, so we won't try and update it when training. 
* Setting collections=[] keeps the variable out of the GraphKeys.GLOBAL_VARIABLES collection used for saving and restoring checkpoints.  
Example: ```input_data = tf.Variable(data_initializer, trainable=False, collections=[])```

## Shared variable
It is possible to reuse weights, just by setting new variable to older one defined previously. For that, you must be in the same namescope, and look for a variable with the same name. Here is an example:
```
def build():
    # Create variable named "weights".
    weights = tf.get_variable("weights", kernel_shape,
        initializer=tf.random_normal_initializer())
    ...

def build_funct():
     with tf.variable_scope("scope1"):
	relu1 = build()
     with tf.variable_scope("scope2"):
	# relu2 is different from relu1, even if they shared the same name, they are in different namescope	
	relu2 = build()

# however calling twice the build_funct() will return an error


result1 = build_funct()
result2 = build_funct()
# Raises ValueError(... scope1/weights already exists ...) because
# f.get_variable_scope().reuse == False 

# to avoid the error, you must defined this way:
with tf.variable_scope("image_filters") as scope:
	result1 = build_funct()
	scope.reuse_variables()
	result2 = build_funct()
```

# How to structure a Tensorflow model
```python
import functools
import tensorflow as tf
from tensorflow.examples.tutorials.mnist import input_data


def doublewrap(function):
    """
    A decorator decorator, allowing to use the decorator to be used without
    parentheses if not arguments are provided. All arguments must be optional.
    """
    @functools.wraps(function)
    def decorator(*args, **kwargs):
        if len(args) == 1 and len(kwargs) == 0 and callable(args[0]):
            return function(args[0])
        else:
            return lambda wrapee: function(wrapee, *args, **kwargs)
    return decorator


@doublewrap
def define_scope(function, scope=None, *args, **kwargs):
    """
    A decorator for functions that define TensorFlow operations. The wrapped
    function will only be executed once. Subsequent calls to it will directly
    return the result so that operations are added to the graph only once.
    The operations added by the function live within a tf.variable_scope(). If
    this decorator is used with arguments, they will be forwarded to the
    variable scope. The scope name defaults to the name of the wrapped
    function.
    """
    attribute = '_cache_' + function.__name__
    name = scope or function.__name__
    @property
    @functools.wraps(function)
    def decorator(self):
        if not hasattr(self, attribute):
            with tf.variable_scope(name, *args, **kwargs):
                setattr(self, attribute, function(self))
        return getattr(self, attribute)
    return decorator


class Model:
    def __init__(self, image, label):
        self.image = image
        self.label = label
        self.prediction
        self.optimize
        self.error
	# define all placeholders here

    @define_scope(initializer=tf.contrib.slim.xavier_initializer())
    def prediction(self):
        x = self.image
        x = tf.contrib.slim.fully_connected(x, 200)
        x = tf.contrib.slim.fully_connected(x, 200)
        x = tf.contrib.slim.fully_connected(x, 10, tf.nn.softmax)
        return x

    @define_scope
    def optimize(self):
        logprob = tf.log(self.prediction + 1e-12)
        cross_entropy = -tf.reduce_sum(self.label * logprob)
        optimizer = tf.train.RMSPropOptimizer(0.03)
        return optimizer.minimize(cross_entropy)

    @define_scope
    def error(self):
        mistakes = tf.not_equal(
            tf.argmax(self.label, 1), tf.argmax(self.prediction, 1))
        return tf.reduce_mean(tf.cast(mistakes, tf.float32))


def main():
    mnist = input_data.read_data_sets('./mnist/', one_hot=True)
    image = tf.placeholder(tf.float32, [None, 784])
    label = tf.placeholder(tf.float32, [None, 10])
    model = Model(image, label)
    sess = tf.Session()
    sess.run(tf.initialize_all_variables())

    for _ in range(10):
      images, labels = mnist.test.images, mnist.test.labels
      error = sess.run(model.error, {image: images, label: labels})
      print('Test error {:6.2f}%'.format(100 * error))
      for _ in range(60):
        images, labels = mnist.train.next_batch(100)
        sess.run(model.optimize, {image: images, label: labels})


if __name__ == '__main__':
    main()
```
# Checkpoint
/TODO
# Tensoarboard
* Launch a session with ```tensorboard --logdir=""```.

### Save a graph
The FileWriter class provides a mechanism to create an event file in a given directory and add summaries and events to it. The class updates is called asynchronously, which means it will never slow down the training loop calling.
```python
sess = tf.Session()
summary_writer = tf.summary.FileWriter('logs', graph=sess.graph)
```
Connection between them is done with the line: ```with tf.Session(graph=graph) as sess:```
### Summary about a data
```
w_h = tf.summary.histogram("weights", W)
```

### Summary about a cost function
```
cost_function = -tf.reduce_sum(var)
# don't need to store the reference, next function is responsible to collect all summaries
tf.summary.scalar("cost_function", cross_entropy)
```

### Merge all summaries operations
```python
merged_summary_op = tf.summary.merge_all()
```

### Collect stat during each iteration
```python
summary_str, _ = sess.run([merged_summary_op, optimize], {x: batchX, y: batchY})
summary_writer.add_summary(summary_str, epoch * nbiters + iter)
```

### Plot embeddings
1. Create an embedding vector (dim: nb_embeddings, embedding_size)
	```python
	embedding = tf.Variable(tf.random_normal([nb_embedding, embedding_size]))
	```
2. Create a tag for every embedding (first name in the file correspond to name of the first embedding
    ```
    LOG_DIR = 'log/'
    metadata = os.path.join(LOG_DIR, 'metadata.tsv')
    with open(metadata, 'w') as metadata_file:
        for name in whatever_object:
            metadata_file.write('%s\n' % name)
    ```
3. Save embedding
    ```
    saver = tf.train.Saver([movie_embedding])
    saver.save(sess, os.path.join(LOG_DIR, 'movie_embeddings.ckpt'))
    ```
4. Create a projector for Tensorboard
    ```
    config = projector.ProjectorConfig()
    embedding = config.embeddings.add()
    embedding.tensor_name = embedding.name # embedding is the tf.Variable()
    embedding.metadata_path = metadata # metadata is a filename
    projector.visualize_embeddings(tf.summary.FileWriter(LOG_DIR), config)
    ```
    
# Regularization
### L2 regularization
```python
w = tf.Variable()
cost = # define your loss
regularizer = tf.nn.l2_loss(w)
loss = cost + regularizer
```
### Dropout
```
hidden_layer_drop = tf.nn.dropout(some_activation_output, keep_prob)
```
### Batch normalization
Batch norm wrapper for two dimensional pre-activation tensor where the mean/variance is calculated over the first axis:
```
def batch_norm_wrapper(inputs, is_training, decay = 0.999):
    scale = tf.Variable(tf.ones([inputs.get_shape()[-1]]))
    beta = tf.Variable(tf.zeros([inputs.get_shape()[-1]]))
    pop_mean = tf.Variable(tf.zeros([inputs.get_shape()[-1]]), trainable=False)
    pop_var = tf.Variable(tf.ones([inputs.get_shape()[-1]]), trainable=False)
    if is_training:
        batch_mean, batch_var = tf.nn.moments(inputs,[0])
        train_mean = tf.assign(pop_mean,
                               pop_mean * decay + batch_mean * (1 - decay))
        train_var = tf.assign(pop_var,
                              pop_var * decay + batch_var * (1 - decay))
        with tf.control_dependencies([train_mean, train_var]):
            return tf.nn.batch_normalization(inputs,
                batch_mean, batch_var, beta, scale, epsilon)
    else:
        return tf.nn.batch_normalization(inputs,
            pop_mean, pop_var, beta, scale, epsilon)
```
and then use the wrapper this way:
```
y = tf.matmul(x,w)
bn = batch_norm_wrapper(y, is_training=True)
```


# Preprocessing
It is possible to load data directly from numpy arrays, however it is best practise to use ```tf.SequenceExample```. It is very verbose, but allows reusability, and really split between the model and the data preprocessing  

1. Create a function to transform a batch element to a ```SequenceExample```:  
    ```python
    def make_example(inputs, labels):
    	ex = tf.train.SequenceExample()
    	# Add non-sequential feature
    	seq_len = len(inputs)
    	ex.context.feature["length"].int64_list.value.append(sequence_length)

    	# Add sequential feature
    	fl_labels = ex.feature_lists.feature_list["labels"]
    	fl_tokens = ex.feature_lists.feature_list["inputs"]
    	for token, label in zip(inputs, labels):
    		fl_labels.feature.add().int64_list.value.append(label)
    		fl_tokens.feature.add().int64_list.value.append(token)
    	return ex
    ```
2. Write all example into TFRecords  
    ```

    import tempfile
    with tempfile.NamedTemporaryFile() as fp:
    	writer = tf.python_io.TFRecordWriter(fp.name)
    	for input, label_sequence in zip(all_inputs, all_labels):
    		ex = make_example(input, label_sequence)
    		writer.write(ex.SerializeToString())
    	writer.close()
    	# check where file is writen with fp.name
    ```
3. Retrieve file with TFRecordReader in an object named ```ex```.
4. Define how to parse the data
    ```python
    context_features = {
        "length": tf.FixedLenFeature([], dtype=tf.int64)
    }

    sequence_features = {
        "tokens": tf.FixedLenSequenceFeature([], dtype=tf.int64),
        "labels": tf.FixedLenSequenceFeature([], dtype=tf.int64)
    }

    context_parsed, sequence_parsed = tf.parse_single_sequence_example(
        serialized=ex,
        context_features=context_features,
        sequence_features=sequence_features
    )
    ```
5. Retrieve the data into array  
    ```python

    # get back in array format
    context = tf.contrib.learn.run_n(context_parsed, n=1, feed_dict=None)
    ```

# Computer vision application



# NLP application
* Look for embedding in a matrix given an id:
```
tf.nn.embedding_lookup(embeddings, mat_ids)
# where embeddings is a tf.Variable()
```

## RNN, LSTM, and shits
LSTM works better than RNN to remenber long term dependencies, because in its form, only the forget gate get multiplied over time, and not activation, hence as long as the forget bias _ft_ is close to 1, it backpropagate up to the time t. More than that, you must make sure to init the bias different than 0!
### Dynamic or static rnn
* Just use ```tf.dynamic_rnn```, it uses a ```tf.While``` allowing to dynamically construct the graph, and passing different sentence lengths between batches.

### Set state for LSTM cell stacked
1. A LSTM cell state contains two tensor (the context, and the hidden state). Let's create a placeholder for both this tensors  
    ```

    # create a (context tensor, hidden tensor) for every layers
    state_placeholder = tf.placeholder(tf.float32, [num_layers, 2, batch_size, state_size])
    # unpack them
    l = tf.unpack(state_placeholder, axis=0)
    ```
2. Transform them into tuples
    ```
    rnn_tuple_state = tuple(
             [tf.nn.rnn_cell.LSTMStateTuple(l[idx][0],l[idx][1])
              for idx in range(num_layers)]
    )
    ```
3. Create the dynamic rnn, and passed initialized state
    ```
    cell = tf.nn.rnn_cell.LSTMCell(state_size, state_is_tuple=True)
    cell = tf.nn.rnn_cell.MultiRNNCell([cell] * num_layers, state_is_tuple=True)

    outputs, state = tf.nn.dynamic_rnn(cell, series_batch_input, initial_state=rnn_tuple_state)
    ```

### Stacking recurrent neural network cells
1. Create the architecture (example for a GRUCell with dropout between every stacking cell
    ```
    from tensorflow.nn.rnn_cell import GRUCell, DropoutWrapper, MultiRNNCell

    num_neurons = 200
    num_layers = 3
    dropout = tf.placeholder(0.1, tf.float32)

    cell = GRUCell(num_neurons)
    cell = DropoutWrapper(cell, output_keep_prob=dropout)
    cell = MultiRNNCell([cell] * num_layers)
    ```

2. Simulate the recurrent network over the time step of the input with ```dynamic_rnn```:
    ```
    output, state = tf.nn.dynamic_rnn(cell, some_variable, dtype=tf.float32)
    ```

### Variable sequence length input

Often, passing sentences to RNN, not all of them are of the same length. Tensorflow wants us to pass into a RNN a tensor of shape ```batch_size x sentence_length x embedding_length```.
To support this in our RNN, we have to first create an 3D array where for each rows (every batch element), we pad with zeros after reaching the end of the batch element sentence. For example if the length of the first sentence is 10, and ```sentence_length=20```, then all element ```tensor[0,10:, :] = 0``` will be zero padded.  

1. It is possible to compute the length of every batch element with this function:  
    ```

    def length(sequence):
    	@sequence: 3D tensor of shape (batch_size, sequence_length, embedding_size)
    	used = tf.sign(tf.reduce_sum(tf.abs(sequence), reduction_indices=2))
    	length = tf.reduce_sum(used, reduction_indics=1)
    	length = tf.cast(length, tf.int32)
    	return length # vector of size (batch_size) containing sentence lengths
    ```
2. Using the length function, we can create our rnn  
    ```

    from tensorflow.nn.rnn_cell import GRUCell

    max_length = 100
    embedding_size = 32
    num_hidden = 120

    sequence = tf.placeholder([None, max_length, embedding_size])
    output, state = tf.nn.dynamic_rnn(
        GRUCell(num_hidden),
        sequence,
        dtype=tf.float32,
        sequence_length=length(sequence),
    )

    ```

There are two cases, whether we are interested by only the last element outputted, or all output at every timestep. Let's define a function for both of them

#### Case 1: output at each timesteps
__Example__: Compute the cross-entropy for every batch element of different size (we can't use ```reduce_mean()```)

```
targets = tf.placeholder([batch_size, sequence_length, output_size])
# targets is padded with zeros in the same way as sequence has been done
def cost(targets):
	cross_entropy = targets * tf.log(output)
	cross_entropy = -tf.reduce_sum(cross_entropy, reduction_indices=2)
	mask = tf.sign(tf.reduce_max(tf.abs(target), reduction_indices=2))
	cross_entropy *= mask

	# Average over all sequence_length
	cross_entropy = tf.reduce_sum(cross_entropy, reduction_indices=1)
	cross_entropy /= tf.reduce_sum(mask, reduction_indices=1)
	return tf.reduce_mean(cross_entropy)
```

#### Case 2: output at the last timestep
__Example__: Get the last output for every batch element:
```
def last_relevant(output, length):
    batch_size = tf.shape(output)[0]
    max_length = tf.shape(output)[1]
    out_size = int(output.get_shape()[2])
    index = tf.range(0, batch_size) * max_length + (length - 1)
    flat = tf.reshape(output, [-1, out_size])
    relevant = tf.gather(flat, index)
    return relevant
```

### Bidirectionnal Recurrent Neural Network
Not so different from the standart ```dynamic_rnn```, we just need to pass cell for forward and backward pass, and it will return two outputs, and two states variables.  d
Example:
```
cell = tf.nn.rnn_cell.LSTMCell(num_units=hidden_size, state_is_tuple=True)
 
outputs, states  = tf.nn.bidirectional_dynamic_rnn(
    cell_fw=cell, # same cell for both passes
    cell_bw=cell,
    dtype=tf.float64,
    sequence_length=X_lengths, # didn't mention them in the snippet
    inputs=X)
output_fw, output_bw = outputs
states_fw, states_bw = states
```
# Graphs
To collect and retrieve vales associated with a graph, it is possible to get them with GraphKeys. For example ```GLOBAL_VARIABLE```, or ```MODEL_VARIABLE```, or ```TRAINABLE_VARIABLE```, ```QUEUE_RUNNERS```, or even more specifically the ```WEIGHTS```, ```BIASES```, or ```ACTIVATIONS```.
* You can get the name of all variables that have not been initialized by passing a list of Variable to the function ```tf.report_uninitialized_variables(list_var). It returns a list of names of uninitialized variables

# Miscellanous
* ```tf.sign(var)``` return -1, 0, or 1 depending the var sign.
* ```tf.reduce_max(3D_tensor, reduction_indices=2)``` return a 2D tensor, where only the max element in the 3dim is kept.
* ```tf.unstack(value, axis=0)```: If given an array of shape (A, B, C, D), and an axis=2, it will return a list of |C| tensor of shape (A, B, D).
* ```tf.nn.moments(x, axes)```: return the mean and variance of the vector in the dimension=axis
* ```tf.nn.xw_plus_b(x, w, b)```: explicit
* tf.global_variables(): return every new variables that are shred across machines in a distributed environment. Each time a Variable() constructor is called, it adds a new variabl ot he graph collection
* tf.convert_to_tensor(args, dtype): (tf.convert_to_tensor([[1, 2],[2, 3]], dtype=tf.float32)): convert an numpy array, a python list or scalar, to a Tensor.
* ```tf.placeholder_with_default(defautl_output, shape)```: One can see a placeholder as an element in the graph that must be fed an output value with the feed dictionnary, however it is possible to define placeholder that take default value.
* ```tf.variable_scope(name_or_scope, default_name)```: if name_or_scope is None, then scope.name is default_name.
* ```tf.get_default_graph().get_operations()```: return all operations in the graph, operations can be filtered by scope then with the python function ```startwith```. It returns a list of tf.ops.Operation

# Tensorflow fold
All tensorflow_fold function to treat sequences:
* td.Map(f): Takes a sequence as input, applies block f to every element in the sequence, and produces a sequence as output.
* td.Fold(f, z): Takes a sequence as input, and performs a left-fold, using the output of block z as the initial element.
* td.RNN(c): A recurrent neural network, which is a combination of Map and Fold. Takes an initial state and input sequence, uses the rnn-cell c to produce new states and outputs from previous states and inputs, and returns a final state and output sequence.
* td.Reduce(f): Takes a sequence as input, and reduces it to a single value by applying f to elements pair-wise, essentially executing a binary expression tree with f.
* td.Zip(): Takes a tuple of sequences as inputs, and produces a sequence of tuples as output.
* td.Broadcast(a): Takes the output of block a, and turns it into an infinite repeating sequence. Typically used in conjunction with Zip and Map, to process each element of a sequence with a function that uses a.

