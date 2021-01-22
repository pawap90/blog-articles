---
title: MongoDB Animated 🍩: Updating elements in arrays
published: false
description: An animated guide on updating elements from embedded arrays in MongoDB, so you don't ever have to google that again.
tags: mongodb, database, array
cover_image: https://i.imgur.com/bsIvu2X.png
series: mongodb-animated
---

Welcome to the second post in my series "MongoDB Animated 🍩", where I provide animated examples and explanations for MongoDB operations that I don't ever want to google again.

- [Introduction](#introduction)
- [Updating elements](#updating-elements)
    - [Updating all elements with $[]](#updating-all-elements-with-)
    - [Updating the first element with $](#updating-the-first-element-with-)
    - [Updating elements that match a filter with $[\<identifier\>]](#updating-elements-that-match-a-filter-with-identifier)
- [Try it yourself](#try-it-yourself)
- [Resources](#resources)
- [Looking for your feedback! 💬](#looking-for-your-feedback-)

# Introduction 

In this post, we'll review different strategies for **updating** elements from embedded arrays.

I'll use the same example collection used in the previous article in this series. It consists of a simple *collection* of "donut combos" from a donut shop. Each combo has a *name* and an *array of donuts* that will be included if the customer chooses that combo. Here's the complete schema:

```js
// donutCombo Schema
{
    name: { type: String, required: true },
    active: { type: Boolean, required: true },
    donuts: [{
        color: { type: String },
        glazing: { type: Boolean }
    }]
}
```

# Updating elements

### Updating all elements with $[]
Using `$set` operator combined with the **all positional operator** `$[]` allows us to update all elements in the array. 

For example, let's say we want to take away the glazing from all donuts in all active documents:

```js
db.donutCombos.updateMany({ active: true }, {  
    $set: {  
        'donuts.$[].glazing': false 
    }  
});
```

// TODO - Gif

Another way to take away the glazing could be by removing the property `glazing` from all donuts in all active documents.
This can be done using the `$unset` operator in combination with the **all positional operator** `$[]`:

```js
db.donutCombos.updateMany({ active: true }, { 
    $unset: { 
        'donuts.$[].glazing': 1 
    } 
});
```
> Notice that, inside the `$unset` object, we must define the key (the property we want to remove) and the value, which doesn't impact the operation. In this case, we used "1" for the value, but you can also use an empty string, for example: `'donuts.$[].glazing': ''`.

### Updating the first element with $
You may need to update only *the first element* of your array. In that case, you should use the **positional operator `$`**. This positional operator will allow us to apply our changes in the first element that matches the `query document` (the first parameter of the `update` method). The array field must be a part of the `query document`.

By combining the `$` positional operator with `$set`, we can update properties from the first array element that matches our `query document`. 

In the following example we will update the *first white donut* in every document and change it's color to "pink".
```js
db.donutCombos.updateMany({ 'donuts.color': 'white' }, { 
    $set: { 
        'donuts.$.color': 'pink'
    } 
});
```

// TODO: Gif

The `$` positional operator can also be combined with `$unset` to remove a property from the first array element that matches our `query document`:

So to try that, let's remove the color from the first white donut found on each document.
```js
db.donutCombos.updateMany({ 'donuts.color': 'white' }, { 
    $unset: { 
        'donuts.$.color': 1 
    } 
});
```

// TODO: Gif

### Updating elements that match a filter with $[\<identifier\>]
To update a set of elements matching certain filters, we must use the **filtered positional operator** `$[<identifier>]`  where `<identifier>` is a placeholder for a value that represents a single element of the array.

We must then use the third parameter (options) of the `updateMany` method to specify a set of `arrayFilters`. Here we will define the conditions each array element we want to update must meet to be updated.

In the next example, we will change every white donut color to green, only in the active documents.

```js
db.donutCombos.updateMany({ active: true }, { 
    $set: { 
        "donuts.$[donut].color": "green" 
    } 
}, 
{ 
    arrayFilters: [{ "donut.color": "white" }] 
});
```

// TO-DO gif

Combining the **filtered positional operator** `$[<identifier>]` (or in our example `$[donut]`) with the operator `$unset`, we can remove a property from all elements in the array that match our `arrayFilter` criteria.

For example, we could remove the color of every white donut in every active document:

```js
db.donutCombos.updateMany({ active: true }, { 
    $unset: { 
        "donuts.$[donut].color": 1
    } 
}, 
{ 
    arrayFilters: [{ "donut.color": "white" }] 
});
```

// TO-DO gif

# Try it yourself

I created a repo to try MongoDB queries in memory using Node.js with Jest and MongoDB Node driver. I use tests to execute the query and verify if everything was correctly updated. I also included a logger that prints the updated documents in the console displaying the changes that were applied using diff highlight syntax:

![The difference of a MongoDB document after being updated](https://i.imgur.com/8yGmVkY.png)

You can find the examples I included in this article in the `tests` folder:

{% github pawap90/try-mongodb-queries %}

# Resources
For more info about updating arrays, here are a few resources from MongoDB's official docs:

- [Array Update Operators (docs.mongodb.com)](https://docs.mongodb.com/manual/reference/operator/update-array/)

- [Update Arrays in a Document (docs.mongodb.com)](https://docs.mongodb.com/drivers/node/fundamentals/crud/write-operations/embedded-arrays)

# Looking for your feedback! 💬
I'm interested in your feedback. Was this post useful? Would you like me to cover any other operation, so you don't *ever* have to google it again?