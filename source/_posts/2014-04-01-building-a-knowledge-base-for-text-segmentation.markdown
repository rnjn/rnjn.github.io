---
layout: post
title: "Building a knowledge base for text segmentation"
date: 2014-04-01 01:02:30 +0530
comments: true
categories: 
---

*Part of a series on extracting information from semi-structured text, starts [here](../03/analysing-semi-structured-text-and-extracting-features.html)*

## Introduction ##

A tedious yet crucial step in the text segmentation approach is building a knowledge base(KB) of known terms and their labels. Labelling (grouping and naming) words is also domain specific, for instance labelling words/phrases for analysis of car sales classified advertisements may be quite different from real estate classified advertisements. Its a cost that is incurred repeatedly as and when new domains are acquired. I present an improvement on the completely manual process below, but first, lets talk about the basic work. As before, examples are from a car buying/selling classified ad corpus.


## KB for text labels ##

A knowledge base is a dictionary of tokens grouped by labels they fall under. We use the term 'label' to define the type of a sub-sequence of tokens, which together join to form a sentence. For instance in the following advertisement

> I want to sell Hundai Santro 2007 model green coloured car in mint condisan. New tyres, regular servicing, good AC and leatherseats. Price Rs 3Lakhs negotiable Contact +91xxxxxxxxxx

the sub-sequence "Hyundai" is actually the "brand" of a vehicle. A KB for domain might therefore look like - 

>
``` python
	knowledge_base = { "brand": ["hyundai", "maruti", "suzuki", "ford"......] }
```

(obviously, there are better data structures to store a KB, this is just conceptual)

Similarly, add words to the KB under "model", "fuel type" and other text labels. Words can belong to more than one label. 


## KB of numbers ##

While words can be compared and classified using a simple dictionary as above, numbers are a completely different story. For instance, to build a KB as above for the label "Price", an infinite number of numbers need to be listed. A Regex might be a better respresentation for the class of numbers. Moreover, a corpus might have a limited number of classes of numeric sequences in tokens and hence we may build a dictionary of Regexes which addresses a class each. In the example domain, the 4 different classes we encounter are
>
* price - 5-7 digits
* year - 4 digits
* phone numbers - 10 digits
* miles/kms driven - 5-7 digits

Writing Regex matchers for these classes is easy - 
>
``` python
        number_matcher = re.compile(r'^\d{5,9}$')
        phone_number = re.compile(r'^\d{10}$')
        year = re.compile(r'^(19|20)\d{2}$')
```

We encounter complexity in mapping numbers to labels when 2 classes of numeric strings overlap. In the example above, "price" and "kms driven" are overlapping classes. To overcome this, we add these steps -
>
* include words like ['kms', 'miles', 'kilometers'....] and ['price', 'cost', 'Rs'....] etc. in the knowledgebases for the two labels.
* create an intermediate label, lets say "number"
* categorise a "number" depending on surrounding words (from the first step)

## Improvements ##

Although the process of building a KB seems simple, it really is not. At the least its time consuming and is a repeated cost everytime a new domain is acquired. Reading and labelling a large corpus of text is also error prone. To this end, an improvement can be to compute vector respresentation of tokens using word2vec[^1] - an unsupervised learning tool. 

word2vec (python interface [gensim](http://radimrehurek.com/gensim/models/word2vec.html)) builds a vocabulary from a corpus and then learns vector representations of words. Vector respresentation is very useful in calculating distance between words, for instance to allow the following (given a lot of relevant text)-

>
vec('Madrid') - vec('Spain') + vec('France') is closer to vec('Paris') than to any other word vector. 

We take their vector representations and use a Eucledian clustering technique like 
[k-means clustering](https://en.wikipedia.org/wiki/K-means_clustering) to identify groups of similar words. Scipy provides a variety of clustering algorithms that might be useful, we choose [k-means](http://docs.scipy.org/doc/scipy/reference/cluster.vq.html) for simplicity and then change depending on feedback.

>word2vec has immense potential and is a tool (and approach) to watch for text analysis solutions. 

The number of dimensions and clusters created are customisable parameters. The resulting output may contain a bunch of clusters which are very similar to the text-only knowledge bases as mentioned above. As an example, with a corpus of around 57000 car buying/selling classified advertisements, with word2vec calculating vector representations in a 100 dimensional space and k-means clustering configured to find out 50 clusters, we were able to find 12 clear clusters for brand names, models, colours, car feature, location of seller etc. These clusters provide an excellent start to builing a knowledge base and cut short the time taken to 'learn' a domain. A larger corpus might help to identify words with high cohesiveness with higher accuracy.

Since a number of clusters identified might not be of value while building our KBs, it would be difficult to automate this approach to remove all manual intervention. Its also important to note that the parameters mentioned above need to tuned and clustering tested using statistical tools to find the best possible combination. Automating this process is not trivial, and is an area of interest and improvement for future.

## Summary ##
Building a KB for a domain involves indentifying and classifying different classes of strings in a corpus of text from said domain. This is a very tedious exercise which requires manual analysis of large volumes of text. Different data structures might be needed to store the KBs, a simple dictionary would do for text, but it needs to be modified to include numbers efficiently, Regex matchers are invaluable. Finally, word2vec provides a definite boost in productivity when 'learning' and new domain and building a KB for it.


[^1]: [word2vec](https://code.google.com/p/word2vec/) : Tool for computing continuous distributes representations of words. A tool to watch.
