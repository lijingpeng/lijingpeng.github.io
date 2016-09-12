title: kaggle之Grupo Bimbo Inventory Demand
date: 2016-09-09 13:48:13
tags: kaggle
category: kaggle
---

## Grupo Bimbo Inventory Demand

[kaggle比赛解决方案集合](https://github.com/lijingpeng/kaggle)

[Grupo Bimbo Inventory Demand](https://www.kaggle.com/c/grupo-bimbo-inventory-demand) 在这个比赛中，我们需要预测某个产品在某个销售点每周的需求量。数据包含墨西哥9周的销售数据。每周，货运车辆把产品发往销售点，每笔交易包含销售量和退货量，其中退货量主要由未销售出的和过期的产品组成。每个产品的需求量是指该商品这周的销售量减去下周的退货量。

几点注意：

1. 测试数据中可能包含训练数据中不存在的商品。这在实际的生活中是十分常见的。所以模型必须很好的适应这一点。  

2. 同一个客户id可能含有不同的客户名字，因为客户名并没有归一化  

3. 需求量是一个大于等于零的值，Venta_uni_hoy - Dev_uni_proxima有时候为负是因为有时候退货量有时候累计了好几星期。  

```
Semana — Week number (From Thursday to Wednesday) - 星期几
Agencia_ID — Sales Depot ID - 销售地id
Canal_ID — Sales Channel ID - 销售渠道id
Ruta_SAK — Route ID (Several routes = Sales Depot)
Cliente_ID — Client ID - 客户id
NombreCliente — Client name - 客户名
Producto_ID — Product ID - 产品id
NombreProducto — Product Name - 产品名
Venta_uni_hoy — Sales unit this week (integer) - 本周销量
Venta_hoy — Sales this week (unit: pesos) - 本周销售额
Dev_uni_proxima — Returns unit next week (integer) - 退货量
Dev_proxima — Returns next week (unit: pesos) - 退货销售额
Demanda_uni_equil — Adjusted Demand (integer) (This is the target you will predict) - 需求量（目标值）
```

销量预测是一个非常有意义的问题，在实际应用中有多种应用价值，例如快递的发货量预测、运输规划等等。粗看起来，这个问题很像一个时间序列的预测问题，但是销售地、人、类目的组合太多，时间累积太短，因此不足以支撑时间序列预测。因此我们把他作为一个回归问题来进行分析。很遗憾的是数据量巨大，解压缩后训练集有7000万+数据，对于我的小破本来说已经无法支撑跑这么大数据集的机器学习任务了。在Kaggle中我找到了一个使用XGBoost的解决方案，下面来简单介绍一下思路和代码。一些关键代码的注释我翻译为中文了。

特征工程的基本思路：

计算第三周的统计特征用来预测第四周的需求、计算第四周的统计特征用来预测第五周的需求...



```python
# A Python implementation of the awesome script by Bohdan Pavlyshenko: http://tinyurl.com/jd6k2kr
# and with inspiration from Rodolfo Lomascolo's  http://tinyurl.com/z6qmxfk
#
# Author: willgraf
import numpy as np
import pandas as pd
import xgboost as xgb
import pdb

from sklearn.metrics import mean_squared_error
from sklearn.cross_validation import train_test_split

from subprocess import check_output

## --------------------------全局变量-----------------------------------------
#
LAG_WEEK_VAL = 3 # set to 3 to use lagged features.
BIG_LAG = True # set to True if to use more than 1 lagged_featc
## --------------------------函数-----------------------------------------

def get_dataframe():
    '''
    载入训练集和测试集，对于训练集我们设置目标为『Demanda_uni_equil』列，测试集目标列暂设定为0.
    这里有一个全局变量LAG_WEEK_VAL，表示week大于LAG_WEEK_VAL的数据作为训练集（训练集中只有第三周到第9周的数据，目标是预测第10和11周的数据）
    '''
    print('Loading training data')
    train = pd.read_csv('train.csv',
                        usecols=['Semana','Agencia_ID','Ruta_SAK','Cliente_ID','Producto_ID','Demanda_uni_equil'])

    print('Loading test data')
    test = pd.read_csv('test.csv',
                       usecols=['Semana','Agencia_ID','Ruta_SAK','Cliente_ID','Producto_ID','id'])

    print('Merging train & test data')
    # lagged week features
    train = train.loc[train['Semana'] > LAG_WEEK_VAL,]
    train['id'] = 0
    test['Demanda_uni_equil'] = None
    train['target'] = train['Demanda_uni_equil']
    test['target'] = 0
    # 将训练集和测试集结合到一起
    return pd.concat([train, test])

def create_lagged_feats(data, num_lag):
    '''
    构建滞后的销量特征。即使用3~8周的销售数据构造特征，为接下来9、10、11周的预测做准备。
    举例来说，第三周的销售数据统计特征作为第四周销量影响因子
    特征列名命名为: target_lNUMBER
    '''
    # 根据这三列生成特征
    keys = ['Semana', 'Cliente_ID', 'Producto_ID']
    lag_feat = 'target_l' + str(num_lag)
    print('Creating lagged feature: %s' % lag_feat)
    data1 = df.loc[: , ['Cliente_ID', 'Producto_ID']]
    # 对week数加1，为接下来的join做准备
    data1['Semana'] = df.loc[: , 'Semana'].apply(lambda x: x + 1)
    # 拷贝训练集中的目标列，并命名为target_lNUMBER，为了接下来求平均值做准备
    data1[lag_feat] = df.loc[: , 'target']
    # groupby 'Semana', 'Cliente_ID', 'Producto_ID'并求target列的平均值
    data1 = pd.groupby(data1, keys).mean().reset_index()
    # JOIN：把平均值的统计特征join到data中
    return pd.merge(data, data1, how='left', on=keys, left_index=False, right_index=False,
                    suffixes=('', '_lag' + str(num_lag)), copy=False)

def create_freq_feats(data, column_name):
    '''
    求列的频率特征，也就是求对应列每周的平均值
    '''
    freq_feat = column_name + '_freq'
    print('Creating frequency feature: %s' % freq_feat)
    # 列+周的target计数
    freq_frame = pd.groupby(data, [column_name, 'Semana'])['target'].count().reset_index()
    freq_frame.rename(columns={'target': freq_feat}, inplace=True)
    # 计算平均值
    freq_frame = pd.groupby(freq_frame, [column_name])[freq_feat].mean().reset_index()

    # 将平均值join回原来的data中
    return pd.merge(data, freq_frame, how='left', on=[column_name], left_index=False,
                    right_index=False, suffixes=('', '_freq'), copy=False)

def build_model(df, features, model_params):
    '''
    构建模型训练和测试
      df: dataframe to be modled - 用来训练和测试的数据
      features: column names of df that should be used in model - 特征列
      params: {xgb_param_key: 'xgb_param_value'} - 参数
    '''
    mask = df['Demanda_uni_equil'].isnull()
    test = df[mask]
    train = df[~mask]
    train.loc[: , 'target'] = train.loc[: , 'target'].apply(lambda x: np.log(x + 1))

    xlf = xgb.XGBRegressor(**model_params)

    x_train, x_test, y_train, y_test = train_test_split(train[features], train['target'], test_size=0.01, random_state=1)

    xlf.fit(x_train, y_train, eval_metric='rmse', verbose=1, eval_set=[(x_test, y_test)], early_stopping_rounds=100)
    preds = xlf.predict(x_test)
    print('RMSE of log(Demanda_uni_equil[x_test]) : %s" ' % str(mean_squared_error(y_test,preds) ** 0.5))

    # 预测第10周的销量
    print('Predicting Demanda_uni_equil for Semana 10')
    data_test_10 = test.loc[test['Semana'] == 10, features]
    preds = xlf.predict(data_test_10)
    data_test_10['Demanda_uni_equil'] = np.exp(preds) - 1
    data_test_10['id'] = test.loc[test['Semana'] == 10, 'id'].astype(int).tolist()

    # 把预测的第10周的销量结果作为特征来预测11周的数据
    print('Creating lagged demand feature for Semana 11')
    data_test_lag = data_test_10[['Cliente_ID', 'Producto_ID']]
    data_test_lag['target_l1'] = data_test_10['Demanda_uni_equil']
    data_test_lag = pd.groupby(data_test_lag,['Cliente_ID','Producto_ID']).mean().reset_index()

    # 预测第11周的销量
    print('Predicting Demanda_uni_equil for Semana 11')
    data_test_11 = test.loc[test['Semana'] == 11, features.difference(['target_l1'])]
    data_test_11 = pd.merge(data_test_11, data_test_lag, how='left', on=['Cliente_ID', 'Producto_ID'],
                            left_index=False, right_index=False, sort=True, copy=False)

    data_test_11 = data_test_11.loc[: , features]#.replace(np.nan, 0, inplace=True)
    preds = xlf.predict(data_test_11)
    data_test_11['Demanda_uni_equil'] = np.exp(preds) - 1
    data_test_11['id'] = test.loc[test['Semana'] == 11, 'id'].astype(int).tolist()

    return pd.concat([data_test_10.loc[: , ['id', 'Demanda_uni_equil']],
                      data_test_11.loc[: , ['id', 'Demanda_uni_equil']]],
                      axis=0, copy=True)

## --------------------------执行-----------------------------------------

## 首先载入训练集和测试集，并进行基本的数据预处理
df = get_dataframe()

## 构建特征
for i in range(1, 1 + 5):
    if not BIG_LAG and i > 1:
        break
    df = create_lagged_feats(df, num_lag=i)

df = df[df['Semana'] > 8]

## 计算列频率特征
for col_name in ['Agencia_ID', 'Ruta_SAK', 'Cliente_ID', 'Producto_ID']:
    df = create_freq_feats(df, col_name)

## 选择部分特征来构建模型
feature_names = df.columns.difference(['id', 'target', 'Demanda_uni_equil'])
print('Data Cleaned.  Using features: %s' % feature_names)

## 模型参数
xgb_params = {
    'max_depth': 10,
    'learning_rate': 0.01,
    'n_estimators': 75,
    'subsample': .85,
    'colsample_bytree': 0.7,
    'objective': 'reg:linear',
    'silent': True,
    'nthread': -1,
    'gamma': 0,
    'min_child_weight': 1,
    'max_delta_step': 0,
    'subsample': 0.85,
    'colsample_bytree': 0.7,
    'colsample_bylevel': 1,
    'reg_alpha': 0,
    'reg_lambda': 1,
    'scale_pos_weight': 1,
    'missing': None,
    'seed': 1
}

## 训练模型并写入结果
submission = build_model(df, feature_names, xgb_params)
submission.to_csv('submission.csv', index=False)
```
