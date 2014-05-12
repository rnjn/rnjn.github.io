---
layout: post
title: "Pretty Python list comprehensions"
date: 2014-05-12 11:40:23 +0530
comments: true
categories: 
---

Python list comprehensions are by far the simplest and most readable loop expressions that I have worked with. Here's an example where I have a list of lists of lists (corpus -> documents -> sentences) where I need to remove some items (called stop_words here) from the sentences.

``` python
corpus = [[[word
            for word in sentence if word not in self._stop_words]
		   for sentence in document]
		 for document in corpus]
														         
```
Notice the array brackets added after each for expression so the program retains the same structure.

Another example, if I have to flatten such a deep list-

``` python
        return [word
		        for document in self._corpus
			   for sentence in document
			 for word in sentence]
```
