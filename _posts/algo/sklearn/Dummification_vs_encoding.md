date: 2016-9-18
title: XGBoost Categorical Variables - Dummification vs encoding
tags:
category: machine learning
---


Question:

When using XGBoost we need to convert categorical variables into numeric.

Would there be any difference in performance/evaluation metrics between the methods of:

dummifying your categorical variables
encoding your categorical variables from e.g. (a,b,c) to (1,2,3)
ALSO:

Would there be any reasons not to go with method 2 by using for example labelencoder?

Answer:

xgboost only deals with numeric columns.

if you have a feature [a,b,b,c] which describes a categorical variable (i.e. no numeric relationship)

Using LabelEncoder you will simply have this:
```
array([0, 1, 1, 2])
```

Xgboost will wrongly interpret this feature as having a numeric relationship! This just maps each string ('a','b','c') to an integer, nothing more.

Proper way

Using OneHotEncoder you will eventually get to this:
```
array([[ 1.,  0.,  0.],
       [ 0.,  1.,  0.],
       [ 0.,  1.,  0.],
       [ 0.,  0.,  1.]])
```

This is the proper representation of a categorical variable for xgboost or any other machine learning tool.

Pandas get_dummies is a nice tool for creating dummy variables (which is easier to use, in my opinion).
