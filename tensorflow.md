# Graph and Session
* A graph defines the computation. It doesn’t compute anything, it doesn’t hold any values, it just defines the operations that you specified in your code.
* A session allows to execute graphs or part of graphs. It allocates resources (on one or more machines) for that and holds the actual values of intermediate results and variables.

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

# Tensoarboard
* Launch a session with ```tensorboard --logdir=""```.

### Save a graph
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
4. Create a projector for tensorboard
```
config = projector.ProjectorConfig()
embedding = config.embeddings.add()
embedding.tensor_name = embedding.name # embedding is the tf.Variable()
embedding.metadata_path = metadata # metadata is a filename
projector.visualize_embeddings(tf.summary.FileWriter(LOG_DIR), config)
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
```python
import tempfile
with tempfile.NamedTemporaryFile() as fp:
	writer = tf.python_io.TFRecordWriter(fp.name)
	for input, label_sequence in zip(all_inputs, all_labels):
		ex = make_example(input, label_sequence)
		writer.write(ex.SerializeToString())
	writer.close()
	# check where file is writen with fp.name
```
3. Retrieve file with TFRecordReader in```ex```.
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

# get back in array format 
context = tf.contrib.learn.run_n(context_parsed, n=1, feed_dict=None)
```
# NLP application
* Look for embedding in a matrix given an id:
```
tf.nn.embedding_lookup(embeddings, mat_ids)
# where embeddings is a tf.Variable()

## RNN, LSTM, and shits
```

