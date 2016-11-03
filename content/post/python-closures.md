+++
date = "2016-11-02T21:37:46+05:30"
sticky = false
tags = ["python"]
title = "Closure in Python"

+++

In Python, a function can be defined inside of another function. This *inner* function has access to variables defined in every function it is nested within. This is called *closure*. Douglas Crockford has said that:

> Closures are the source of enormous expressive power.

However, Crockford had said this about closures in *Javascript: The Good Parts*, his excellent introduction to the Javascript language. While closures in Python work in much the same way, and offer many of the same benefits, there are a few surprising differences. Also, a lot has changed in the way the language deals with closures between Python 2 and 3.

### Table of Contents

* [Closure in Python 2](#closure-python-2)
  * [Under the hood](#closure-python-2-internals)
  * [Simulating writeable closure in Python 2](writeable-closure-python-2)
* [Closure in Python 3](#writeable-closure-python-3)


### <a name="closure-python-2"></a>Closure in Python 2

In *Python 2.x*, the outer function's variables are made available inside the inner function's closure as long as these variables aren't modified by the inner function:

{{< highlight python >}}
def outer_v1():
    msg = "Help! I'm wrapped inside a closure!"
    def inner():
        print msg
    inner()
    print msg
    return inner
{{< /highlight >}}

Invoking the outer function prints the following to the console:

```
>>> outer_v1()
Help! I'm wrapped inside a closure!
Help! I'm wrapped inside a closure!
```

However, the moment you try to modify the value of the variable, it escapes the closure and becomes a *new* variable that's scoped to the inner function:

{{< highlight python >}}
def outer_v2():
    msg = "I'm scoped to the outer function."
    def inner():
        msg = "I'm scoped to the inner function."
        print msg
    inner()
    print msg
    return inner
{{< /highlight >}}

Invoking the outer function now has the following result:

```
>>> outer_v2()
I'm scoped to the inner function.
I'm scoped to the outer function.
```

Even after the inner function has been invoked, the value of the outer function's `msg` variable remains unchanged. This is an indication that the variable `msg` used inside the inner function was created afresh with the inner function as its scope.

#### <a name="closure-python-2-internals"></a>Under the hood

Though it's possible to guess all of this by looking at the output from the two function calls above, we can get a much better feel for what's going on by inspecting the *special* `__closure__` property of the inner function. Later, we will use the `dis` module to generate opcode and get far more satisfying results. For instance, in the case of the first function `outer_v1`, we can inspect the inner function's closure like so:

{{< highlight objdump >}}
>>> i = outer_v1()
>>> i.__closure__
(<cell at 0x10ae106e0: str object at 0x10ae13b70>,)
>>> i.__closure__[0].cell_contents
"Help! I'm wrapped inside a closure!"
{{< /highlight >}}

We see that the string `"Help! I'm wrapped inside a closure!"` is quite literally wrapped as a tuple element inside the inner function's `__closure__` property. Next, let's take a look at the opcode for `outer_v1`:

{{< highlight objdump >}}
>>> from dis import dis
>>> dis(outer_v1)

2           0 LOAD_CONST               1 ("Help! I'm ... closure!")
            3 STORE_DEREF              0 (msg)

3           6 LOAD_CLOSURE             0 (msg)
            9 BUILD_TUPLE              1
           12 LOAD_CONST               2 (<code object ... line 3>)
           15 MAKE_CLOSURE             0
           18 STORE_FAST               0 (inner)

5          21 LOAD_FAST                0 (inner)
           24 CALL_FUNCTION            0
           27 POP_TOP

6          28 LOAD_DEREF               0 (msg)
           31 PRINT_ITEM
           32 PRINT_NEWLINE

7          33 LOAD_FAST                0 (inner)
           36 RETURN_VALUE
{{< /highlight >}}

The key instructions to watch out for, are `6 LOAD_CLOSURE 0` and `15 MAKE_CLOSURE 0`. According to the Python docs, the [LOAD_CLOSURE](https://docs.python.org/2/library/dis.html#opcode-LOAD_CLOSURE) instruction adds a value to the free variable storage array (`co_freevars`) of the inner function, and  [MAKE_CLOSURE](https://docs.python.org/2/library/dis.html#opcode-MAKE_CLOSURE) then assigns the list of all loaded closure variables to the inner function's `__closure__` property. These instructions are missing from the opcode of the second version of our outer function, `outer_v2`:

{{< highlight objdump >}}
>>> from dis import dis
>>> dis(outer_v2)

2           0 LOAD_CONST               1 ("I'm scoped ... function.")
            3 STORE_FAST               0 (msg)

3           6 LOAD_CONST               2 (<code object ... line 3>)
            9 MAKE_FUNCTION            0
           12 STORE_FAST               1 (inner)

6          15 LOAD_FAST                1 (inner)
           18 CALL_FUNCTION            0
           21 POP_TOP

7          22 LOAD_FAST                0 (msg)
           25 PRINT_ITEM
           26 PRINT_NEWLINE

8          27 LOAD_FAST                1 (inner)
           30 RETURN_VALUE
{{< /highlight >}}

And as you would expect in this case, the `__closure__` property of the inner function is set to `None`:

{{< highlight objdump >}}
>>> i = outer_v2()
>>> i.__closure__ is None
True
{{< /highlight >}}

#### <a name="writeable-closure-python-2"></a>Simulating writeable closure in Python 2

Python's `dict` data structure can be employed to simulate a writeable closure in Python:

{{< highlight python >}}
def outer_v3():
    closure = { 'counter': 0 }
    def inner():
        closure['counter'] += 1
    for i in xrange(5):
        inner()
    print "inner() was invoked {n} times".format(n=closure['counter'])
{{< /highlight >}}

Invoking the outer function prints the following to the console:

{{< highlight objdump >}}
>>> outer_v3()
inner() was invoked 5 times
{{< /highlight >}}

As you might have guessed, this is due to the fact that we can update the key-value pairs in-place inside the `closure` dictionary variable without assigning it an entirely different dictionary object. In other words, we're able to achieve this because [Python dictionaries are mutable](https://docs.python.org/2/library/stdtypes.html#mapping-types-dict), while [Python strings are immutable](https://docs.python.org/2/library/stdtypes.html#typesseq-mutable).

### <a name="writeable-closure-python-3"></a>Closure in Python 3

In Python 3, the `nonlocal` qualifier can be used to inform the compiler that a variable is already bound to an outer function's scope, and should not be created afresh. Because of this, it becomes possible to update closure variables without enclosing them inside a dictionary.

{{< highlight python >}}
def outer_v4():
    counter = 0
    def inner():
        nonlocal counter  # `counter` is present in an outer scope
        counter += 1
    for i in range(5):
        inner()
    print("inner() was invoked {n} times".format(n=counter))
{{< /highlight >}}
