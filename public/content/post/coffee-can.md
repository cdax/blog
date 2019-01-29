+++
date = "2014-10-31T14:16:01+05:30"
sticky = false
tags = ["puzzles"]
title = "The Coffee Can Problem"

+++

#### The Problem

>Imagine you've got a can full of coffee beans -- some are black and the rest are white. Outside of this can, you've also got yourself a large reservoir of black coffee beans. You keep picking out beans from your can, two at a time. If both the beans you picked are the same colour, you've got to throw both of them away and add to the can with a black bean from your stockpile. On the other hand, if the beans are different colours, you've got to throw the black bean away and restore the white one to the can. Keep repeating this till you're left with exactly one bean.

The *Coffee Can Problem* is two-fold. First, you've got to prove that this series of experiments <sub>will</sub> terminate with exactly one bean left in the can. And second, you need to determine the colour of the last remaining bean given the respective numbers of black and white beans that you started out with (say b<sub>1</sub> and w<sub>1</sub>).

#### The Solution

Let's say that just before the i-th pick, you're left with exactly b<sub>i</sub> black beans and w<sub>i</sub> white beans inside the can. The i-th pick can have one of three possible outcomes:

##### Outcome I: Both the beans you picked were BLACK

If this happens, then after throwing away both beans and replacing them with one black bean from your pile, you're left with b<sub>i</sub> - 1 black beans and w<sub>i</sub> white beans.

##### Outcome II: Both the beans you picked were WHITE

In this situation, you've got to throw away the two white beans and replace them with a black bean. So that you're left with b<sub>i</sub> + 1 black beans and w<sub>i</sub> - 2 white beans.

##### Outcome III: The beans you picked were different colours

In this last scenario, you'll be throwing away the black bean and putting the white one back in the can, so that you've got yourself b<sub>i</sub> - 1 black beans and w<sub>i</sub> white beans, just like *Outcome I*.

Two observations are key:

* No matter what the outcome, the total number of beans in the can will *always* come down by one.
* The number of white beans in the can will either stay the same, or drop by two, or in other words, white beans always leave the can two at a time.

From the first observation, it directly follows that you'll always finish with exactly one bean in the can (because each pick takes a bean away from the total).

And putting both observations together, it is easy to see that you'll run out of white beans if and only if you had an even number of white beans to begin with. *It doesn't matter how many black beans you had in the can at any point of time.*

#### Thinking about Corner Cases

The reasoning above assumes that at least one pick will always be made, so a corner case occurs when you only get one bean to begin with. *Does our solution still hold?*

#### Takeaways and Source

My biggest learning from this problem was about the existence of white coffee beans. Apparently, it's an under-roasted variety that results in a lighter-than-usual brew.

The *Coffee Can Problem* originally appeared in David Gries's *Science of Programming*, though I found it among the exercise problems in Jon Bentley's *Programming Pearls* (Ch. 4 - Writing Correct Programs).
