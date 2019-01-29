+++
date = "2014-12-28T13:01:57+05:30"
sticky = false
tags = ["mongodb"]
title = "MongoDB: Selecting Documents at Random"

+++

#### The Problem

While implementing the back-end for [Mathbin](http://math-bin.herokuapp.com/), I wanted an API endpoint that would return a random document from a Mongo collection. A very naive way of approaching this would be to select all documents and then picking the document at a random index in the returned array.

Now, I don't expect Mathbin's database to hit big numbers anytime soon, but this really got me thinking about how this problem could be solved efficiently for collections having millions, maybe billions of documents.

#### The Solution

There's a nice solution that makes retrieval very efficient at the price of some extra storage. The idea is to associate a random value with each document while inserting it into the collection. This can be achieved in Mongoose using a numeric field with a default value of `Math.random()`

Later, during retrieval, a simple filter gets the job done:

{{< highlight javascript >}}
var randomDoc = Collection.findOne().gte(Math.random());
{{< /highlight >}}


#### A Better Solution

The problem with the solution above is that if the collection has very few documents, there's a very good chance that the filter might return no document at all. *What are the odds that the random value generated in the query will be less than at least one document?* Not very good when the number of documents is too little. So how do we make this better?

If the *greater-than-or-equals* query fails, simply flip it around:

{{< highlight javascript >}}
//Uses Mongoose promises to pass values from one async call to the next  
var pivot = Math.random();  
Collection.findOne().gte(pvt).exec()  
.then(function(snippet) {
    //If the gte filter returned nothing, try an lt filter!
    if(snippet === null)
        return Collection.findOne().lt(pvt).exec();
    else return snippet;
}).then(function(snippet) {
    //At this point, snippet contains a random document from the collection.
}).error(console.log)
.end();
{{< /highlight >}}

And we're set! The only way this could fail now is if the Collection is empty to begin with -- an easy edge case to handle. Let me know what you think in the comments.
