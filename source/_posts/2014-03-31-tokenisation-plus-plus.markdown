---
layout: post
title: "Tokenisation++"
date: 2014-03-31 18:16:59 +0530
comments: true
categories: 
---

*Part of a series on extracting information from semi-structured text, starts [here](../03/analysing-semi-structured-text-and-extracting-features.html)*

## Introduction ##

> corpus - a collection of written texts, especially the entire works of a particular author or a body of writing on a particular subject.

Given a corpus of text, the first step is to [tokenise](https://en.wikipedia.org/wiki/Tokenize) it. To elaborate, I would take an example of a corpus of classified advertisements for selling and buying cars. The added work on top of regular tokenisation addresses wayward spellings and sentence formations common in the Indian web-based classified ads corpora. Subsequent work in analysing the text would work on the tokens we obtain. I assume that you have a list of strings, each an advertisement. I used python to code this, you will see references to python libraries that have been helpful. 

A typical classified advertisement selling a car follows

> I want to sell Hundai Santro 2007 model green coloured car in mint condisan. New tyres, regular servicing, good AC and leatherseats. Price Rs 3Lakhs negotiable Contact +91xxxxxxxxxx

The ordering of steps below help me explain the approach. The code shouldn't follow logical order, it wouldn't be performant. 

## Regular tokenisation ##

We start with breaking the advertisement down into a list of sentences. Then we break each sentence down into a list of words. A typical approach for English would be break sentences down by finding one of {';', ':', ',', '.', '!', '?'} and then break them down further by using spaces. However, different languages have different ways of ending a sentence or separation between words. There's further complexity if we include local flavour or contexts for e.g. a period between two numbers is not a full-stop. Instead of mulling through different variations, we can delegate this work to the nltk [PunktSentenceTokeniser](http://www.nltk.org/api/nltk.tokenize.html#nltk.tokenize.punkt.PunktSentenceTokenizer), which is trained on a collection of English text. Our task it then reduced to the following

``` python
	            for ad in ads:
                    sentences = sent_tokenize(document)
        			for sentence in sentences:
        				tokens = word_tokenize(sentence)
```

## The ++ ##

Now, it makes sense to look at the corpus in general and create some general rules about tokens, which boost accuracy of subsequent analysis. This step might also be called a data cleaning step. Here are a few that come in handy for the example domain -

for text input 
>
* all tokens should be in lower case
* all tokens would represent individual words, not combinations
* all tokens would contain human readable unicode
* no periods in between a word, unless its numeric

for numerical input 
>
* all numerical input will be in numeric characters (e.g. convert 3Lakhs to 300000)
* no commas in numerical input
* no special characters in a phone number string

We logically craft these in {before/during/after} the sentence and word tokenisation. Many of the above, like the numerical token rules can be performed easily using Regex matches and substitutions. These rules may not be premeditated. Indeed, we depend on the subsequent analysis process to for feedback on the tokenisation process to update the rules. For instance, if we encounter a situation where spelling variations and mistakes lower accruacy of analysis, we might add the following - 

>
* use only one way to spell a word
* try best to correct spellings

Now, there are different ways to spell correct. A very simple approach is to use a language dictionary in conjunction with following process for domain specific words -
>
* create a [histogram](https://en.wikipedia.org/wiki/Histogram) of all words. The Counter class in python [collections](https://docs.python.org/2/library/collections.html) is a fast and easy way to do this.
>
	``` 
		word_histogram = Counter(tokens)
	```
* if a word exists in the dictionary, don't do anything.
* for each word not in the dictionary, iterate over the histogram and find words which are closest in terms of [Levenshtein distance](https://en.wikipedia.org/wiki/Levenshtein_distance). Use a threshold number for choosing the list of candidates, to improve accuracy.
* find the most frequently used word in the above list and replace the less frequently used one.
* insert this into a secondary dictionary for future use.

The Enchant library ([pyenchant](http://pythonhosted.org/pyenchant/) interface for python) provides a great interface to an English dictionary. Notably, it allows for new words to be added and it suggests words given an incorrect one. 

This approach depends on the assumption that there are more correctly spelled words than not. Unfortunately, sometimes this is not correct and analysis algorithms may have to address this problem, or as a final solution manual correction may be required. There are other customisations that might help, like choosing a different distance measure to fit the domain of words and errors. In a non-English locale a phonetic similarity measure might help, for instance to figure that 'condisan' above is 'condition'.

The example advertisement copied above showcases two more issues - misspelling a proper noun (Hyundai v/s Hundai) and joined word (leather seats v/s leatherseats). For the former, when using Enchant, its advisable to add a list of proper nouns to the enchant dictionary, so as to improve suggestions. We also add another step of checking distance from this list of proper nouns (which might contain brand names etc.). The latter however, involves a performance intensive solution of creating smaller sets of words from a bigger word (if its not found in our customised dictionary) and then checking if in a particular combination of these sets, a high number of smaller words are found in our customised dictionary. In the case above, we might want to limit our sets to [bigrams](https://en.wikipedia.org/wiki/Ngram) first-
> [{lea, therseats}, {leat, herseats}, {leath, erseats}, {leathe, rseats}, {leather, seats}.....]

and clearly the set {leather, seats} is a strong contender. In case we don't find a meaningful set from bigrams, we create trigrams and so on.


## Stop words and other measures ##

As a penultimate step, we remove from our list of tokens, words which have negative or no significant effect on the final outcome of text analysis. We keep a list of such [stop words](https://en.wikipedia.org/wiki/Stop_words) and when we encounter them, we remove them from our token list. An example of such a list can be -

>
a,able,about,across,after,all,almost,also,am,among,an,and,
any,are,as,at,be,because,been,but,by,can,cannot,could,dear,
did,do,does,either,else,ever,every,for,from,get,got,had,has,
have,he,her,hers,him,his,how,however,i,if,in,into,is,it,its,
just,least,let,like,likely,may,me,might,most,must,my,neither,
no,nor,not,of,off,often,on,only,or,other,our,own,rather,said,
say,says,she,should,since,so,some,than,that,the,their,them,then,
there,these,they,this,tis,to,too,twas,us,wants,was,we,were,what,
when,where,which,while,who,whom,why,will,with,would,yet,you,your

Each problem domain has its own set of stop words, once again results of our next steps in analysis will help building a better list. For e.g., viz. the automobile related classified advertisements, the following may be considered stop words
> model,showroom,fast,comfort,trend...

Finally, we choose a threshold token size/word length, below which a token doesn't qualify to be part of the analysis steps. For the example domain, upto 2 letter words are not significant, and either hurt performance or accuracy. We also remove all remaining letters that are not numeric/alphabetic.

## Output ##

Example output for the example advertisement may be -

>["sell", "hyundai", "santro", "2007", "green", "mint", "condition", "new", "tyres", regular", "servicing", "good", "leather", "seats", "price", "300000", "negotiable", "contact", "91xxxxxxx"]

## Summary ##

As part of the tokenisation and cleaning process, we break text into a list sentences each of which is a list of words (called tokens). We remove stop/insignificant words, and we code in a bunch of rules that help our analysis process, for instance - dealing with numbers etc. We craft this tokenisation process based on experience, then fit it into a solution and gather feedback, and then use this feedback to update the process. Rinse, repeat.
