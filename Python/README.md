
# The Hitchhiker’s Guide to Python


## 1. Writing Great Python Code
#### Structure of the repository
How to structure a python repo:
```{python}
README.rst
LICENCE
setup.py
requirements.txt
sample/__init__.py
sample/core.py
sample/helpers.py
docs/
tests/
```
* sample: module package where the core focus of the repository
* makefile: example of a python makefile
```{bash}
init:
    pip install -r requirements.txt
test:
    py.test tests
```

##### How to make test file import your packaged module to test it:
1. Create a file context.py in the tests folder. Insert  ``` sys.path.insert(0, os.path.abspath('..'))```
2. Within the individual test modules, import the module like so ```{python} from .context import sample```



## Structure of the code
When facing circular dependency, one solution is to import statements inside methods or functions


## Modules
An import ``` import modu``` will look for the proper file in the same directory first, then in the path of the Python interpreter, and raise an _ImportError_ if it is not found.  
Once found, the module is executed in an isolated scope. Function and class definitions are stored in the module's dictionnary. It's different from C++ code, where code is put with the caller's code. So there is no worry to override an existing function with the same name.  
__From worst to best:__



```python
from numpy import * # JUST BAD
from numpy import sqrt # And if we redefined sqrt in our code...
import numpy # better
```

## Packages
Any directory with an ```__init__.py__``` file is considered a Python package. A module in the pack/ folder can then be imported calling ```import pack.modu```.

## Object-oriented programming
Everything is an object in Python. Some people prone to use Python using only stateless functions, because it is a better programming paradigm for concurrency.


## Decorators


```python

def decorator(func):
    # manipulate
    return func

foo = decorator(foo)

# is equivalent to

@decorator
def foo():
    # do something
# foo won't be called without being passed by the decorator function
```

## Context manager


```python
with open('file.txt') as f:
    contents = f.read()
# it ensure that f is closed at some point. 
```

There are one easy ways to implement this functionnality yourself. It is using the generator approach using Python's own _contextlib_


```python
from contextlib import contextmanager

@contextmanager
def custom_open(filename):
    f = open(filename)
    try:
        yield f
    finally
        f.close()

# and then
with custom_open('file.txt') as f:
    contents = f.read()
```

## Dynamic typing
In Python, variables are not a segment of the computer's memory where some value is written, they are tags, or names posting to the object. The dynamic typing of Python is often considered to a be weakness, because something name 'a' can be set to many different things.  
There is no interest to reuse the same variable name for different things.


## Mutable and imutable types
* Mutable types : lists, dictionnaries

It is better to append to a list of char, than append to a string


```python
nums = ""
for n in range(20):
  nums += str(n) # BAD
```

## 2. Code stype
* Pass argument in order to a function, to avoid having to precise which argument it is.
* Kwargs and args uses:


```python
def func(required_arg, *args, **kwargs):
    # required_arg is a positional-only parameter.
    print required_arg

    # args is a tuple of positional arguments,
    # because the parameter name has * prepended.
    if args: # If args is not empty.
        print args

    # kwargs is a dictionary of keyword arguments,
    # because the parameter name has ** prepended.
    if kwargs: # If kwargs is not empty.
        print kwargs
    print('\n')

# func() : wrong, take at least, one argument

func("required argument")

func("required argument", 1, 2, '3')

func("required argument", 1, 2, '3', keyword1=4, keyword2="foo")

```

    required argument
    
    
    required argument
    (1, 2, '3')
    
    
    required argument
    (1, 2, '3')
    {'keyword2': 'foo', 'keyword1': 4}
    
    


## Unpacking


```python
for index, item in enumerate(some_list):
    # do something
```

## Indentation

Respect the standart established by pep8
```{bash}
pip install pep8 
pep8 main.py # return all error of indentation in a file

pip install autopep8
pep8 --in-place main.py # transform the file on place
```

## Short ways to manipulate lists

* Return all elements in an array


```python
b = filter(lambda x: x > 4, a)
```

## C module imported in Python


```python
import ctypes
# load a shared library, that has been created with
# 1. g++ -shared -o library.so -fPIC file.C
lib = ctype.cdll.LoadLibrary('./library.so')

function = lib.foo # a function foo is in the library
function.restype = None

# Create matrix object
doublecpp = ndpointer(ctypes.c_double, ndim=2, flags="C_CONTINUOUS")

# define function parameters
function.argtyptes[doublecpp, ctypes.c_int, ctypes.c_int]

# call the function
function(np.ones((3, 3)), 3, 3)
```
