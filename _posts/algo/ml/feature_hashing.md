date: 2015-9-25
title: Feature hashing
tags: 
category: machine learning
---

In machine learning, feature hashing, also known as the hashing trick(by analogy to the kernel trick), is a fast and space-efficient way of vectorizing features, i.e. turning arbitrary features into indices in a vector or matrix. It works by applying a hash function to the features and using their hash values as indices directly, rather than looking the indices up in an associative array.

### Motivating example
In a typical document classification task, the input to the machine learning algorithm (both during learning and classification) is free text. From this, a bag of words (BOW) representation is constructed: the individual tokens are extracted and counted, and each distinct token in the training set defines a feature (independent variable) of each of the documents in both the training and test sets.

Machine learning algorithms, however, are typically defined in terms of numerical vectors. Therefore, the bags of words for a set of documents is regarded as a term-document matrix where each row is a single document, and each column is a single feature/word; the entry i, j in such a matrix captures the frequency (or weight) of the j'th term of the vocabulary in document i. (An alternative convention swaps the rows and columns of the matrix, but this difference is immaterial.) Typically, these vectors are extremely sparse.

The common approach is to construct, at learning time or prior to that, a dictionary representation of the vocabulary of the training set, and use that to map words to indices. Hash tables and tries are common candidates for dictionary implementation. E.g., the three documents

John likes to watch movies.
Mary likes movies too.
John also likes football.
can be converted, using the dictionary
```
Term    Index
John    1
likes   2
to      3
watch   4
movies  5
Mary    6
too     7
also    8
football9
```
to the term-document matrix

```
1 & 1 & 1 & 1 & 1 & 0 & 0 & 0 & 0 
0 & 1 & 0 & 0 & 1 & 1 & 1 & 0 & 0 
1 & 1 & 0 & 0 & 0 & 0 & 0 & 1 & 1
```

(Punctuation was removed, as is usual in document classification and clustering.)

The problem with this process is that such dictionaries take up a large amount of storage space and grow in size as the training set grows. On the contrary, if the vocabulary is kept fixed and not increased with a growing training set, an adversary may try to invent new words or misspellings that are not in the stored vocabulary so as to circumvent a machine learned filter. This difficulty is why feature hashing has been tried for spam filtering at Yahoo! Research.

Note that the hashing trick isn't limited to text classification and similar tasks at the document level, but can be applied to any problem that involves large (perhaps unbounded) numbers of features.

### Feature vectorization using the hashing trick
Instead of maintaining a dictionary, a feature vectorizer that uses the hashing trick can build a vector of a pre-defined length by applying a hash function h to the features (e.g., words) in the items under consideration, then using the hash values directly as feature indices and updating the resulting vector at those indices:

 function hashing_vectorizer(features : array of string, N : integer):
```
     x := new vector[N]
     for f in features:
         h := hash(f)
         x[h mod N] += 1
     return x
```
It has been suggested that a second, single-bit output hash function ξ be used to determine the sign of the update value, to counter the effect of hash collisions. If such a hash function is used, the algorithm becomes

 function hashing_vectorizer(features : array of string, N : integer):
```
     x := new vector[N]
     for f in features:
         h := hash(f)
         idx := h mod N
         if ξ(f) == 1:
             x[idx] += 1
         else:
             x[idx] -= 1
     return x
```
The above pseudocode actually converts each sample into a vector. An optimized version would instead only generate a stream of (h,ξ) pairs and let the learning and prediction algorithms consume such streams; a linear model can then be implemented as a single hash table representing the coefficient vector.

