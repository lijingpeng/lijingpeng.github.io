title: kaggle之数字序列预测
date: 2016-08-19 13:48:13
tags: kaggle
category: kaggle
---

## 数字序列预测

[Github地址](https://github.com/lijingpeng/kaggle/tree/master/competitions/Sequence)     
[Kaggle地址](https://www.kaggle.com/c/integer-sequence-learning)


```python
# -*- coding: UTF-8 -*-
%matplotlib inline

import pandas as pd
import string
import numpy as np
import matplotlib.pyplot as plt
from sklearn import preprocessing
```


```python
train = pd.read_csv('train.csv')
test = pd.read_csv('test.csv')
```


```python
last = test.Sequence.apply(lambda x: pd.Series(x.split(','))).mode(axis=1).fillna(0)

```

```python
submission = pd.DataFrame({'Id': test['Id'], 'Last': last[0]})
submission.to_csv('mode.csv', index=False)
```

提交Kaggle之后是0.05680
