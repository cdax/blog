+++
date = "2014-11-27T13:28:06+05:30"
sticky = false
tags = ["algorithms", "data structures"]
title = "Binary Indexed Trees"

+++

>"A problem that seems difficult may have a simple, unexpected solution." Unlike the advanced methods, the aha! insights of algorithms don't come only after extensive study; they're available to any programmer willing to think seriously before, during and after coding.

*Jon Bentley (quoting Martin Gardner), in the column Aha! Algorithms from his book Programming Pearls*

#### The Origin of BITs

The Binary Indexed Tree is hands down the most elegant data structure that I've had the pleasure of studying. It offers an embarrassingly simple yet highly efficient solution to the problem of maintaining cumulative object frequencies.

Imagine you're supposed to keep track of the number of times each symbol appears in a long string, knowing the complete universe of possible symbols (or the *symbol alphabet*). Two operations are key - `UPDATE` the frequency of a symbol as it is read from the string, and `QUERY` the frequency of a symbol at any given time.

A third kind of operation becomes important when a meaningful order can be imposed on the symbol alphabet, like say if the alphabet was a list of numbers that can be sorted. Here, it might be useful to track the cumulative frequency or `COUNT` of symbols less than a given symbol *x*.

The Binary Indexed Tree solves each of the `UPDATE`, `QUERY` and `COUNT` problems in O(M) time, and using O(n) space where n is the cardinality of the symbol alphabet, and M is just the number of set bits (1s) in n's binary representation!

BITs were first described in a 1994 paper titled [A New Data Structure for Cumulative Frequency Tables](http://pdf.aminer.org/001/073/976/a_new_data_structure_for_cumulative_frequency_tables.pdf) by Peter Fenwick, and are also known by their eponymous name, Fenwick Trees.

#### The Aha! Insight

The unexpected search technique used by BITs can be better understood by drawing parallels with the way many people learn how to convert from binary to decimal notation. We use the following lookup table:

<table>  
    <tbody><tr>
        <th><strong>Bit position, i</strong></th>
        <td>0</td>
        <td>1</td>
        <td>2</td>
        <td>...</td>
        <td>n</td>
    </tr>
    <tr>
        <th><strong>Value, v<sub>i</sub></strong></th>
        <td>1</td>
        <td>2</td>
        <td>4</td>
        <td>...</td>
        <td>2<sup>n</sup></td>
    </tr>
</tbody></table>

and the decimal representation is simply the sum of the values related with the positions of the set bits. For 13, for example:

<p>13 = (1101)<sub>2</sub> = v<sub>3</sub> + v<sub>2</sub> + v<sub>0</sub> = 8 + 4 + 1</p>

The Fenwick Tree is an array analogous to the lookup table above, where each of the array elements si, is a carefully computed partial frequency sum (a sub-frequency). If fi is the actual frequency of the ith element, and r is the position of the least significant bit in i (for example, r1 = 0, r2 = 1, r3 = 0, and so on), then

<p>s<sub>i</sub> = f<sub>i - 2<sup>r</sup> + 1</sub> + f<sub>i - 2<sup>r</sup> + 2</sub> + ... + f<sub>i</sub></p>

#### Three key observations

* Note that the element frequencies that make up s<sub>i</sub> and s<sub>i - 2<sup>r</sup></sub> are disjoint sets.
* Note also, that s<sub>0</sub> = f<sub>0</sub>
* Finally, note that any frequency f<sub>j</sub> appears in each of the sums s<sub>i</sub> where j ≥ i - 2<sup>r</sup> + 1

Talking in terms of bits, i - 2<sup>r</sup> can be computed by stripping away the least significant bit from i. For more on bit-twiddling, take a look at this [useful reference guide](https://graphics.stanford.edu/~seander/bithacks.html).

#### Implementing `COUNT` and `QUERY`

##### `COUNT`
The cumulative frequency up until the ith element, COUNT<sub>i</sub> is defined as the sum of the actual frequencies of all elements less than or equal to the ith. COUNT<sub>i</sub> can be calculated from the Fenwick Tree by observing that:

<p>COUNT<sub>i</sub> <br>
= f<sub>0</sub> + ... + f<sub>i - 1</sub> + f<sub>i</sub><br>
= f<sub>0</sub> + ... + f<sub>i - 2<sup>r</sup> - 1</sub> + f<sub>i - 2<sup>r</sup></sub> + (f<sub>i - 2<sup>r</sup> + 1</sub> + f<sub>i - 2<sup>r</sup> + 2</sub> + ... + f<sub>i</sub>)<br>
= s<sub>0</sub> + ... + s<sub>i - 2<sup>r</sup></sub> + s<sub>i</sub></p>

In order to compute COUNT<sub>i</sub>, we start with the sub-frequency at index i and at every step add a new sub-frequency after stripping away i's least significant bit. This continues till i runs out of set bits, i.e., till i becomes equal to zero. For example,

<p>COUNT<sub>13</sub> = s<sub>13</sub> + s<sub>12</sub> + s<sub>8</sub> + s<sub>0</sub></p>

##### `QUERY`
Retrieving the actual frequency of the element at index *i* is as simple as calculating COUNT<sub>i</sub> and COUNT<sub>i - 1</sub> and computing the difference. Both `QUERY` and `COUNT` are thus equally efficient.

Never forget that `QUERY` can be made even faster by identifying the longest common suffix in the binary representations for *i* and *i - 1*. (How does this help?)
