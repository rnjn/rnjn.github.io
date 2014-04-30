---
layout: post
title: "Building a positional model"
date: 2014-04-03 12:38:28 +0530
comments: true
categories: 
---

*Part of a series on extracting information from semi-structured text, starts [here](../03/analysing-semi-structured-text-and-extracting-features.html)*

## Introduction ##

Blocking and labelling using known words stored in a knowledge base lets us extract known information from a document. However, for words that are new to the knowledge base, like a new car brand or model in the example domain, blocking and labelling fails. To be able to categorise or label new words, we take a probabilistic approach. Given a sizeable number of documents, we build a positional model, that helps us find an appropriate label for a new word given its position in the document.

## Preparing data ##

Before we start calculating probabilities, we aggregate a sizeable corpus of documents, lets say 50000 or more, and run our blocking and labelling step. We then search for documents which have "unknown" blocks and remove them from our set. We do this assuming these are < 10% of the corpus. If this is not true, the quality of the knowledge base needs to be improved, and we keep iterating the cycle below
> improve KB -> Block and Label > % of documents not labelled completely > improve KB...

till we reach this goal.

Once we pass this step, we move on to building a representative matrix of documents with each cell denoting the position of a label. For a labelled document like below -
> ["sell", "hyundai", "santro", "2007", "green", "tyres",  "leather", "seats", "price", "300000", "contact", "91xxxxxxx"]

the row vector (of labels by position) in the matrix that denotes this document looks like
> ["ad_type", "brand", "model", "year", "colour", "features", "price", "contact"]

Now, to reduce the size of the matrix in memory, we may represent each label by a unique number. and so our representative row vector looks like
> [2,3,4,5,6,7,8,9]

(we reserve 1 for "unknown")

The length of each document may differ, so we find the longest document, and we pad all documents with 0's to fit the matrix width. An example matrix of 10 documents across 10 labels may look like -

``` python
[[ 5, 10,  6,  3,  8,  7,  3,  3, 11,  6],
[ 6,  6,  5,  5,  8,  6,  2,  9, 10,  8],
[ 6,  2,  4,  5,  9,  7,  9,  5,  5,  7],
[ 4,  8,  9,  7,  4,  6,  6,  8,  5,  2],
[ 5,  5,  4, 10,  8,  8,  2, 11, 10,  9],
[10,  9,  9,  6, 11,  9,  9,  8,  8,  6],
[ 7,  2, 11,  3,  9,  4,  7, 11,  8,  3],
[ 5,  5,  5,  8,  8,  2, 11,  7, 11, 10],
[ 9, 11,  3,  2,  9,  4,  5,  6,  3,  8],
[ 5,  8,  8,  8, 11,  7,  7,  7,  3,  6]]
```

where each row represents a document and each column represents a position in the document, and numbers denote a label.

## Positional Model ##

To build a vector for positional probabilities for a label we follow

>
``` python
for each label:
	for each position:
		probability[position] = sum(occurence of this label)/number of documents
```
Over 50000 documents and 12 labels, with a maximum size of 150 tokens, traditional looping does not perform well. We turn to the [numpy](http://www.numpy.org) python library to help us crunch these numbers faster for us.

First, we derive a matrix for each label, which represents the position of this label only across documents. We call this matrix the label sieve. For the matrix above, choosing label 5, the sieve should look like -

``` python
[[1, 0, 0, 0, 0, 0, 0, 0, 0, 0],
[0, 0, 1, 1, 0, 0, 0, 0, 0, 0],
[0, 0, 0, 1, 0, 0, 0, 1, 1, 0],
[0, 0, 0, 0, 0, 0, 0, 0, 1, 0],
[1, 1, 0, 0, 0, 0, 0, 0, 0, 0],
[0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
[0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
[1, 1, 1, 0, 0, 0, 0, 0, 0, 0],
[0, 0, 0, 0, 0, 0, 1, 0, 0, 0],
[1, 0, 0, 0, 0, 0, 0, 0, 0, 0]]
```

where 1 denotes the presence of label 5 in a document, and 0 denotes absence.

The following code does this for us

``` python
        label_sieves = [(matrix == n) * 1 for n in labels]

```

> numpy matrices can be queried like -> matrix == n to result in a boolean matrix for the presence of n. the result * 1 just converts this to the above.

Now its easy for us to sum all column vectors in the matrix above, where each column represents a position in the document. Again, numpy provides faster calculations, as below

``` python
        label_position_totals = [numpy.sum(label_sieve, axis=0)
                                 for label_sieve in label_sieves]

```

and then we just divide by the count of documents.

Our final positional matrix may look like -

``` python
 [[ 0. ,  0.2,  0. ,  0.1,  0. ,  0.1,  0.2,  0. ,  0. ,  0.1],
 [ 0. ,  0. ,  0.1,  0.2,  0. ,  0. ,  0.1,  0.1,  0.2,  0.1],
 [ 0.1,  0. ,  0.2,  0. ,  0.1,  0.2,  0. ,  0. ,  0. ,  0. ],
 [ 0.4,  0.2,  0.2,  0.2,  0. ,  0. ,  0.1,  0.1,  0.2,  0. ],
 [ 0.2,  0.1,  0.1,  0.1,  0. ,  0.2,  0.1,  0.1,  0. ,  0.3],
 [ 0.1,  0. ,  0. ,  0.1,  0. ,  0.3,  0.2,  0.2,  0. ,  0.1],
 [ 0. ,  0.2,  0.1,  0.2,  0.4,  0.1,  0. ,  0.2,  0.2,  0.2],
 [ 0.1,  0.1,  0.2,  0. ,  0.3,  0.1,  0.2,  0.1,  0. ,  0.1],
 [ 0.1,  0.1,  0. ,  0.1,  0. ,  0. ,  0. ,  0. ,  0.2,  0.1],
 [ 0. ,  0.1,  0.1,  0. ,  0.2,  0. ,  0.1,  0.2,  0.2,  0. ]]
```


## Extras ##

We may derive a different model if we calculate the probability as -

>
``` python
for each label:
	for each position:
		probability[position] = sum(occurence of this label)/**sum(occurance of any label other than 0)**
```

To make this change, we first build a sieve which denotes the presence of all labels, and then sum column-wise to get a denominator vector by position.
``` python
        sieve = (matrix > 0) * 1.0
        denominators = numpy.sum(sieve, axis=0).tolist()[0]

```

Our final probability calculation looks like
``` python
	probabilities = [numpy.divide(label_totals, denominators)
        for label_totals in label_position_totals]


```
where numpy.divide does an item-wise division.

This calculation produces skewed numbers for a column where the presence labels in the column is very low. To normalise the probabilities in this case, the previous method is better.

## Performance ##

Traditional looping method to build this model for around 50000 documents, 14 labels and a maximum size of 144 words (50000 X 144 size matrix), takes around 2 seconds with our modest python skills. Using numpy, as coded above, we could do this in milli-seconds. The added advantage being expressive code.
