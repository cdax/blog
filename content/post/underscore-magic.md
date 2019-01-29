+++
date = "2014-12-26T13:42:48+05:30"
sticky = false
tags = ["javascript"]
title = "Underscore Magic!"

+++

Front-end developers should know [Underscore.js](http://underscorejs.org/) like the backs of their hands. It brings jQuery's philosophy of *Write less, do more* to the area of array/object manipulation. It's almost like a shim for some really useful functions that JavaScript should be supporting natively. No wonder then, that it finds itself listed as a dependency by many popular client-and server-side projects like *Backbone.js* and *Ghost*.

![](/img/underscore.png)

In this post, I'm going to talk about three cool ways in which I've seen Underscore (or its cousin [Lo-Dash](http://lodash.com/)) put to use.

#### Working with Request Parameters

I noticed these tricks while going through the source of *Ghost*. Ghost server has a number of API endpoints, each with a set of blacklisted/whitelisted request parameters. So what happens when someone tries to meddle with the API calls made by the client-side app? Something like this:

{{< highlight javascript >}}
//pick() returns a copy of the object, filtered to only have values for the whitelisted keys (or array of valid keys)  
req.params = _.pick(req.params, whitelistedParams);  
{{< /highlight >}}

...or this:

{{< highlight javascript >}}
//omit() returns a copy of the object, filtered to omit the blacklisted keys (or array of keys)  
req.params = _.omit(req.params, blacklistedParams);  
{{< /highlight >}}

Similarly, in order to normalize/standardize a request by adding in default values for required parameters, simply do:

{{< highlight javascript >}}
//defaults() fills in undefined properties in object with the first value present in the following list of defaults objects  
req.params = _.defaults(req.params, defaultRequest);
{{< /highlight >}}

#### Chaining

When `chain()` is called on an array/object, Underscore attaches its own methods to it, returning a *wrapped object*. The beauty of `chain()` is that the following two lines of code become equivalent:

{{< highlight javascript >}}
//Imperative style  
req.params = _.pick(req.params, whitelistedParams);  
//Functional style
req.params = _.chain(req.params)  
                .pick(whitelistedParams)
                .value();
{{< /highlight >}}

The call to `value()` at the end strips off underscore methods from the wrapped object returned by the previous method in the chain. Here's another example of this *functional* way of writing JavaScript:

{{< highlight javascript >}}
req.params = _.chain(req.params)  
                .keys()
                .size()
                .value();
                .value();
{{< /highlight >}}

#### Templating

Underscore also comes with an inbuilt templating method that parses and populates Embedded JS templates, taking values from a passed-in object. In fact, this is a fairly common pattern in Backbone.js:

{{< highlight javascript >}}
//Defining the template  
var viewTemplate = _.template(  
                    "<p>Hi! I'm <%= firstName %>" +
                    " and I'm <%= age %> " +
                    "<% if(age === 1) { %>" +
                        "year" +
                    "<% } else { %>" +
                        "years" +
                    "<% } %> old.</p>"
                  );
//Populating the template
$(".view").append(viewTemplate({
    firstName: "Chitharanjan",
    lastName: "Das",
    age: 24
});
{{< /highlight >}}

#### Read the Docs!

Underscore can be used to achieve a lot more fancy-ness than can be put into a couple of blogposts. To find out more, head on over to [the docs](http://underscorejs.org/).
