+++
date = "2014-10-06T14:45:24+05:30"
sticky = false
tags = ["algorithms"]
title = "Fast Modular Exponentiation"

+++

#### The Problem

... is to compute *b<sup>e</sup> (mod m)*, where *b*, *e*, and *m* are positive integers.

*e.g.,* for *b = 2*, *e = 109* and *m = 10<sup>9</sup> + 7*,

*b<sup>e</sup> (mod m) = 140625001*

#### A Naive Solution

... would be one that first computes *b<sup>e</sup>* and then computes its remainder *mod m*.

Speaking in terms of the basic arithmetic operations, this method would require *O(e)* multiplications, and a lot of space to store the computed value of *b<sup>e</sup>*.

Consider the example given above where *b* is 2 and *e* is 10<sup>9</sup>. Storing the computed value of *b<sup>e</sup>* in this case will require (10<sup>9</sup> + 1) bits! Clearly, such a solution will not cut it with large values of *b*, *e* and *m*.

#### Improving the space bound

The costly *O(log<sub>2</sub> b<sup>e</sup>)* bound on space can be improved significantly if, instead of computing the remainder at the end, we compute remainders after each multiplication, using the property

*{ p × q } (mod r) = \[ p × { q (mod r) } \] (mod r)*

Plugging in values for *b*, *e* and *m* from the example above, we begin multiplying as follows:

*b (mod m) = 2 (mod m) = 2*

*b<sup>2</sup> (mod m) = \[ b × { b (mod m) } \] (mod m) = \[ 2 × 2 \] (mod m) = 4*

*b<sup>3</sup> (mod m) = \[ b × { b<sup>2</sup> (mod m) } \] (mod m) = \[ 2 × 4 \] (mod m) = 8*

...and so on, till

*b<sup>e</sup> (mod m) = \[ b × { b<sup>e - 1</sup> (mod m) } \] (mod m)*

This solution runs in *O(e)* time, using *O(log<sub>2</sub> (b × m))* space. The space improvement is significant for large values of *e*.

Here's a sketch for a recursive function based on the algorithm above, taken from a Wikipedia article:

```
function modular_pow(b, e, m)  
    c := 1
    for e_prime = 1 to e
        c := (c * b) mod m
    return c
```

#### Improving the time bound

Now that we've sufficiently improved the worst-case space bound, let's do something about the *O(e)* time bound, which can be troublesome for large values of *e*. Consider the binary representation for the exponent *e*:

*e = ( a<sub>0</sub> × 2<sup>0</sup> ) + ( a<sub>1</sub> × 2<sup>1</sup> ) + ... + ( a<sub>n</sub> × 2<sup>n</sup>  )*, where *n = log<sub>2</sub> e*

Thus,

*b<sup>e</sup> (mod m) = b<sup>( a<sub>0</sub> × 2<sup>0</sup> ) + ( a<sub>1</sub> × 2<sup>1</sup> ) + ... + ( a<sub>n</sub> × 2<sup>n</sup>  )</sup> (mod m)*

*b<sup>e</sup> (mod m) = { b<sup>( a<sub>0</sub> × 2<sup>0</sup> )</sup> (mod m) } × { b<sup>( a<sub>1</sub> × 2<sup>1</sup> )</sup> (mod m) } × ... × { b<sup>( a<sub>n</sub> × 2<sup>n</sup> )</sup> (mod m) }*

*b<sup>e</sup> (mod m) = { (b<sup>2<sup>0</sup></sup> )<sup>a<sub>0</sub></sup> (mod m) } × { (b<sup>2<sup>1</sup></sup> )<sup>a<sub>1</sub></sup> (mod m) } × ... × { (b<sup>2<sup>n</sup></sup> )<sup>a<sub>n</sub></sup> (mod m) }*

At this point, two observations are key:

* When *a<sub>i</sub> = 0*, the term *{ (b<sup>2<sup>i</sup></sup>)<sup>a<sub>i</sub></sup> (mod m) }* reduces to 1
* When *a<sub>i</sub> = 1*, the same term becomes equal to *b<sup>2<sup>i</sup></sup> (mod m)*, which can be computed iteratively as *{ b<sup>2</sup> (mod m) × b<sup>2<sup>(i - 1)</sup></sup> (mod m) } (mod m)*

This leaves us with a fast and space-efficient algorithm running in only *O(log<sub>2</sub> e)* time and requiring only *O(log<sub>2</sub> (b × m))* space. Here's a sketch of the complete algorithm:

```
function modular_pow(b, e, m)  
    result := 1
    b := b mod m
    while e > 0
        if (e mod 2 == 1):
           result := (result * b) mod m
        e := e >> 1
        b := (b * b) mod m
    return result
```
