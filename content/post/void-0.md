+++
date = "2014-12-25T13:53:53+05:30"
sticky = false
tags = ["javascript"]
title = "Javascript's \"void\" operator"

+++

Being a relative newcomer to JavaScript, I figured a good way to learn the community's best practices would be to spend some time looking under the hood of popular open-source projects that I use. Lately I've been working with the excellent and lightweight *Backbone.js*, so I thought its [annotated source](http://backbonejs.org/docs/backbone.html) would be the perfect place to start.

After ten minutes of reading, I was staring at this puzzling snippet of code on my screen, which at first glance seems to be a check for `null`/`undefined`.

{{< highlight javascript >}}
if (options.parse === void 0) options.parse = true;
{{< /highlight >}}

#### The Plot Thickens

To my eyes, void 0 looked like an open invitation to learn another bit of JavaScript trivia. I was on to [MDN](http://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/void) and [Stack Overflow](http://stackoverflow.com/questions/7452341/what-does-void-0-mean) in a flash.

Here's what MDN's got to say about `void`:

>The `void` operator evaluates the given expression and then returns `undefined`

*Aha!* So that line above was indeed a check for `undefined`. Try it out! Open up your browser's console and type in the following...

{{< highlight javascript >}}
>> void 0;  
>> void "Hello, world.";  
>> void (1 + 2);
{{< /highlight >}}

...to see your browser say `undefined`, `undefined`, `undefined`!

But wait, what about all those other ways of checking whether a variable is undefined. How is `x === void 0` better than any of those? What about `x === undefined` or `typeof x === "undefined"`?

#### `undefined` is just another variable

My most surprising learning from this was that `undefined` (in some old browsers) is just another global variable that can be overridden. If you're using any of these old browsers, try this:

{{< highlight javascript >}}
alert(undefined = "Not undefined")  
{{< /highlight >}}

In a world where `undefined` is mutable and could easily be overridden by you or any of your dependencies, `x === undefined` is not a reliable check. `typeof x === "undefined"` still remains a pretty solid candidate, however. For more JavaScript gotchas like the mutability of undefined, head on over to [wtf.js](http://wtf.js/)
