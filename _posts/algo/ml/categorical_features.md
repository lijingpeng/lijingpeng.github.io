date: 2016-9-18
title: Handling categorical features
tags:
category: machine learning
---


## Handling categorical features: get_dummies, OneHotEncoder and multicollinearity

Get_dummies is a common way to create dummy variables for categorical features. While it is widely used, there are some drawbacks. First, it modifies your dataframe. When you have a categorical feature with hundreds of categories, get_dummies adds hundreds of dummy variables to the dataframe. And you may need to drop the categorical feature after creating dummies if you want to quickly assign features to X (independent variables). Another thing about get_dummies is, if there are many categorical features in the dataset, you will have to do get_dummies for every feature. Even constructing a loop to do so is a lot code to write.  

If you do not want to mess up your data frame, or you are too lazy to write too much code (like me), OneHotEncoder is for you. OneHotEncoder is a sklearn preprocessing function. Unlike get_dummies, OHE does not add variables to your data frame. It creates dummy variables by transforming X, and all the dummies are stored in X. And you can specifiy which columns you want to create dummies when you fit X by OHE.  

However, there are also some cons about OHE. First, OHE only works with categories that are integers over or equal to zero, which means if the category names are strings or negative numbers, you will have to do some transformation before fitting OHE.  

Secondly, you may not be able to drop one dummy variable in order to avoid multicollinearity after fitting OHE as there are no separate variables in the dataframe. One way to address this problem is to use regularization (Lasso) with the model to drop redundant dummies. As there are two dummies which provide the exact same information, one of them must be dropped when minimizing the regularization term in the cost function.  

A final con about OHE is that, you may not be able to tell which dummies are left and which are dropped. So, if you are doing bottom up feature selection, get_dummies may be a better choice as you can tell which features are selected and reassign X. Or if you are trying to identify the single effect of a certain dummy on y, get_dummies is the choice.

From: https://medium.com/@guaisang/handling-categorical-features-get-dummies-onehotencoder-and-multicollinearity-f9d473a40417#.o5hqnz95x
