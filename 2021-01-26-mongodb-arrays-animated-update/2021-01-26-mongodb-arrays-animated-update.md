---
title: MongoDB Animated: Adding and removing elements from arrays
published: false
description: An animated guide on how to add and remove elements from embedded arrays in MongoDB
tags: mongodb, database, array
cover_image: https://i.imgur.com/PXY3KWT.png
series: mongodb-animated
---

Last week I had to work on an old project with MongoDB. A few documents with _**embedded arrays**_ needed to be updated, and I realized that every time I have to do this sort of operations, I end up googling a lot to re-learn things I forgot.

So this time I decided to take thorough notes of everything I learn and write an article with examples so future Paula can go directly to the examples instead of googling everything all over again. And hey! Maybe someone else finds it useful as well.

I also included short animations illustrating each example 🤓

- [Introduction](#introduction)
- [Adding new elements into the array](#adding-new-elements-into-the-array)
    - [Adding a single element at the end](#adding-a-single-element-at-the-end)
    - [Adding a single element into a specific position](#adding-a-single-element-into-a-specific-position)
    - [Adding multiple elements](#adding-multiple-elements)
- [Removing an element from the array](#removing-an-element-from-the-array)
- [Try it yourself](#try-it-yourself)
- [Resources](#resources)

# Introduction 

In this article I'm only going to talk about **adding** and **removing** elements from documents with embedded arrays. I will be posting a new article next week about how to update the contents of elements in the array.

The example DB we are going to use consists of a simple *collection* of "donut combos" from a donut shop. Each combo has a *name* and an *array of donuts* that will be included if the customer chooses that combo. Here's the complete schema:

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

# Adding new elements into the array

We can add a new element to an array using the `$push` operator. By default, the new element will be added at the end of the array.

### Adding a single element at the end

In the following example, we will add a pink donut to the first document found with `name` "No Choco".

```js
db.donutCombos.updateOne({ name: "No Choco" }, {
    $push: {
        donuts: {
            color: "pink",
            glazing: true
        }
    }
});
```

![MongoDB finds the document where the name is "No Choco" and adds a pink donut at the end of the donuts array](https://i.imgur.com/WiskZLy.gif)

### Adding a single element into a specific position

Using the `$position` modifier, we can specify where in the array we want our new elements to be positioned. In order to use the `$position` modifier, we also need to use the `$each` modifier, even though we are only adding a single element.

In the following example, we will add a pink donut to the first document found with `name` "No Choco", in the *2nd position *of the array.

```js
db.donutCombos.updateOne({ name: "No Choco" }, {
    $push: {
        donuts: {
            $position: 2,
            $each: [{
                color: "pink",
                glazing: true
            }]
        }
    }
});
```

![MongoDB finds the document where the name is "No Choco" and adds a pink donut at position 2 of the array](https://i.imgur.com/rir1Aaq.gif)

### Adding multiple elements

Using the modifier `$each` we can push more than one element into our array.

In the following example, we will add two donuts (white and chocolate) at the end of the array in the first document with name "B&W".

```js
db.donutCombos.updateOne({ name: "B&W" }, {
    $push: {
        donuts: {
            $each: [
                { color: "white", glazing: true },
                { color: "chocolate", glazing: true }
            ]
        }
    }
});
```

![MongoDB finds the document where the name is "B&W" and adds a white donut and a chocolate donut at the end of the array](https://i.imgur.com/WsjeU4e.gif)

# Removing an element from the array

To remove an element from the array we use the operator `$pull`. Inside the $pull object we must specify a key-value pair: the key is the name of the array property from our document and the value is the filter we want to apply to define which elements should be removed.

In the following example, we will remove all the *white* donuts from the *active* documents in the *donutCombos* collection.

```js
db.donutCombos.updateMany({ active: true }, {
    $pull: {
        donuts: { color: "white" }
    }
});
```
![MongoDB finds all active documents removes all the white donuts](https://i.imgur.com/QkLvWUG.gif)

# Try it yourself

I created a repo to try MongoDB queries in memory using Node.js with Jest and MongoDB Node driver. I use tests to execute the query and verify if everything was correctly updated. I also included a logger that prints the updated documents in the console displaying the changes that were applied using diff highlight syntax:

![The difference of a MongoDB document after being updated](https://i.imgur.com/8yGmVkY.png)

You can find the examples I included in this article in the `tests` folder:

{% github pawap90/try-mongodb-queries %}


# Resources
For more info about updating arrays here are a few resources from MongoDB's official docs:

- [Array Update Operators (docs.mongodb.com)](https://docs.mongodb.com/manual/reference/operator/update-array/)

- [Update Arrays in a Document (docs.mongodb.com)](https://docs.mongodb.com/drivers/node/fundamentals/crud/write-operations/embedded-arrays)