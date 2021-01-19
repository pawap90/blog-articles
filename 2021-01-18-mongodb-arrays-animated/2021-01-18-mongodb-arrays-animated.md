---
title: MongoDB Animated: Updating elements in arrays
published: false
description: An animated guide on how to update elements from embedded arrays in MongoDB
tags: mongodb, database, array
cover_image: https://i.imgur.com/bsIvu2X.png
series: mongodb-animated
---

// TO-DO

- [Introduction](#introduction)
- [Updating elements](#updating-elements)
- [Try it yourself](#try-it-yourself)
- [Resources](#resources)

# Introduction 

In this article I'm only going to talk about **updating** elements from embedded arrays.

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
// TODO


# Try it yourself

I created a repo to try MongoDB queries in memory using Node.js with Jest and MongoDB Node driver. I use tests to execute the query and verify if everything was correctly updated. I also included a logger that prints the updated documents in the console displaying the changes that were applied using diff highlight syntax:

![The difference of a MongoDB document after being updated](https://i.imgur.com/8yGmVkY.png)

You can find the examples I included in this article in the `tests` folder:

{% github pawap90/try-mongodb-queries %}


# Resources
For more info about updating arrays here are a few resources from MongoDB's official docs:

- [Array Update Operators (docs.mongodb.com)](https://docs.mongodb.com/manual/reference/operator/update-array/)

- [Update Arrays in a Document (docs.mongodb.com)](https://docs.mongodb.com/drivers/node/fundamentals/crud/write-operations/embedded-arrays)