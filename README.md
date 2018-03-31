# Pythonic

Use Python form within node!

Pythonic is designed for prototyping apps with data-driven Python backends. It is not recommended for scalable or production-ready apps – for that, you might want to look at a micro-service architecture.

This package can be used along with [Pythonic-Client](https://github.com/ideo-colab/pythonic-client) to quickly make Python functionality available on the fronted.

## Why?

Python is the language of choice for data science and machine learning (as well as other applications), and a node stack is an excellent choice for prototyping highly-interactive apps. Pythonic makes it straightforward to take advantage of both of these strengths simultaneously.

## Basic Usage

Pythonic tries to mimics how you'd use Python in, well, Python.

So say we have this Python module:
`fancycomputation.py`
```py
def my_very_fancy_function(first, second, commentary='nice job!'):
  # ... something fancy you need python for ...
  return str(first + second) + ' commentary'
```

In another Python module, after setting a `PYTHONPATH` to that module, we could call it like this:
```py
from fancycomputation import my_very_fancy_function

result = my_very_fancy_function(1,2, commentary='hooray!')

print(result)
# 3 hooray!
```

Using Pythonic, we can do essentially the same thing in node:
```js
const Pythonic = require('pythonic');

const py = Pythonic({
  path: './python-stuff', // equivalent to setting PYTHONPATH
  imports: [{
    from: 'fancycomputation',
    import: 'my_very_fancy_function',
  }],
});

// callback for when python has started
py._.onReady(() => {
  py.my_very_fancy_function([1, 2], {commentary: 'way to go!'})
  .then((result) => {
    console.log(result)
    // 3 way to go!
  })
});
```

## Docs

### Pythonic(_{options}_)
Returns a `Pythonic` instance, starts a python kernel and attaches callables in node as described in options.

### Options

*`path`* Array|String
The path or paths to append to the `PYTHONPATH`

*`imports`* Array as `{import, [from]}`
Describes which Python modules to import. Supports these patterns from Python:
```py
from MODULE import OBJECT1, OBJECT2
# {from: 'MODULE' import: ['OBJECT1', 'OBJECT2']}
from PACKAGE import MODULE
# {from: 'PACKAGE', import: 'MODULE'}
import MODULE
# {import: 'MODULE'}
from MODULE import *
# {from: 'MODULE', import: '*'}
```


### Calling imported Python functions

All imports are attached to the `Pythonic` instance as they would be to the global namespace in Python. Only callables are made available to Node (not constants).

All calls to Python functions return a promise that resolves to the result.

So, for example:
```js
const py = Pythonic({
  imports:[
    {from: 'my_mod', import: ['my_func', 'my_other_func']}
  ]
  })
```
will make `my_func` and `my_other_func` available like so:
```js
  py.my_func().then(result => {
    console.log(result)
  })
  py.my_other_func().then(result => {
    console.log(result)
  })
```


Similarly if `my_other_mod` contains `do_this()` and `do_that()`, we can run:
```js
const py = Pythonic({
  imports:[
    {import: 'my_other_mod'}
  ]
  })
```
and now we'll be able to do this:
```js
  py.my_other_mod.do_this().then(result => {
    console.log(result)
  })
  py.my_other_mod.do_that().then(result => {
    console.log(result)
  })
```

#### Handing arguments to Python
Since JavaScript doesn't have a notion of Keyword Arguments, instead you can make calls to Python that contain arguments like so:

```js
  py.my_function([args], {kwargs})
```
You may omit either `[args]` or `{kwargs}` if the function you're calling doesn't require them.


#### Instantiating Python classes
Say you've imported a Python class:
```js
const py = Pythonic({
  imports:[
    {from: 'my_mod', import: 'MyClass'}
  ]
  })
```
Pythonic allows you to create and use an instance of that class:
```js
py._.initClass(
  {
    class: 'MyClass',
    as: 'mc',
    args: [/*init args*/],
    kwargs: {/*init kwargs*/}
  }
)
// the initClass method returns a promise
.then(()=>{
  // once the class is init'ed we can call it:
  py.mc.instace_method(['good stuff']).then(result => {
    console.log(result)
  })
})
```
The instance will continue to be available as `py.mc` with all of it's callable methods attached.


### Methods

*`_.onReady(_callback_)`*
Attach a callback function to call when the instance of Pythonic is ready.

*`_.initClass(_{options}_)`*
Instantiate a python class and attach it to the instance of Pythonic. Returns a Promise. See [Instantiating Python Classes](#instantiating-python-classes) above.

*`_.importModules(_[modules]_)`*
Import modules after the initial init. Follows the same pattern as in the [init options](#options).

#### What's with the \_?
Since the `py.` namespaces is reserved for the python modules imported by the user, instance methods on the `Pythonic` object are proxied to `py._.`.