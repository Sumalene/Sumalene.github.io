---
layout: single
title: "[AI]预测课程销售"
categories: [Web, AI]
last_modified_at: 2023-08-01
excerpt: "Playground Series - Season 3， Episode 19"
header: 
  teaser: https://uploadstatic.mihoyo.com/bh3-wiki/2023/03/20/75216984/2204a9e06f9c1ef5e79c3a1cc3d7fe92_7511506494193485957.png
---

# Forecasting Mini-Course Sales
> 深夜打榜.感谢歪果老哥的激情指点,Long live Open Source

## 数据集描述
对于这个挑战，将预测来自不同（真实！）国家/地区的不同虚构 Kaggle 品牌商店的各种虚构学习模块的全年销售额。该数据集是完全合成的，但包含您在真实世界数据中看到的许多效果，例如周末和假日效应、季节性等。您的任务是预测 2022 年的销售额。

---
比赛链接：[playground-series-s3e19](https://www.kaggle.com/competitions/playground-series-s3e19/)

我的源码：[March7th_OpenEDA](https://github.com/Sumalene/AI_EDA/blob/main/Playground/Forecasting%20Mini-Course%20Sales.ipynb)

## EDA
![image](https://github.com/Sumalene/Sumalene.github.io/assets/124686994/6cfebe34-7a14-459d-bff9-84ce0d5f3cc3)
![image](https://github.com/Sumalene/Sumalene.github.io/assets/124686994/4c7297f6-2f3e-4d0d-b992-fc79fc5bbb90)
![image](https://github.com/Sumalene/Sumalene.github.io/assets/124686994/3eb93a11-5812-4a23-8ee4-8ca33386fcef)
![image](https://github.com/Sumalene/Sumalene.github.io/assets/124686994/80287d9d-ca48-49f5-a04c-f6a743404b24)

趋势很好分析。关键在于异常年份的处理。

## 关键数据处理

```python
# 分割日期 将日期分割为天、星期、周、月和年，然后使用这些特征创建这些特征的周期版本。
train['day'] = train['date'].dt.day # 从日期中提取天
train['dayofweek'] = train['date'].dt.dayofweek # 从日期中提取星期
train['week'] = train['date'].dt.isocalendar().week # 从日期中提取周
train['month'] = train['date'].dt.month # 从日期中提取月
train['year'] = train['date'].dt.year # 从日期中提取年

# 修复新冠年 这一年可能会造成很多过度拟合。从某种意义上说，2020 年的销售分布使其成为一个离群值，因此我们将通过使其与邻近年份相似来解决这个问题。
# 计算非疫情年份（2019 年和 2021 年）的平均月销量。
avg_sales_non_virus = train[train['year'].isin([2019, 2021])].groupby('month')['num_sold'].mean()

# 计算疫情年份（2020 年）的平均月销量。
avg_sales_virus = train[train['year'] == 2020].groupby('month')['num_sold'].mean()

# 计算每个月的比率。
ratios_for_months = avg_sales_non_virus / avg_sales_virus

# 将 2020 年每个行的 num_sold 列乘以该行月份的对应比率。
train.loc[train['year'] == 2020, 'num_sold'] *= train['month'].map(ratios_for_months)
```
![image](https://github.com/Sumalene/Sumalene.github.io/assets/124686994/9f3379d9-9ab9-438b-8b62-268f294c4f1b)

> 时间具有周期性。例如，1 月 1 日与 12 月 31 日很接近。因此，我们可以将日期特征分解为正弦和余弦分量，以反映这种周期性。正弦分量表示周期性特征的上升部分，余弦分量表示周期性特征的下降部分。这样做可以帮助机器学习模型更好地理解时间特征的影响。例如，如果我们知道 1 月 1 日通常会比其他日期有更多的销售，那么我们可以预期模型会将正弦分量与销售数量的正相关性。

之后添加周末，节假日特征即可

## 建模

没有意外，直接使用.CatBoost是一种基于对称决策树（oblivious trees）为基学习器实现的参数较少、支持类别型变量和高准确性的GBDT框架，主要解决的痛点是高效合理地处理类别型特征，这一点从它的名字中可以看出来，CatBoost是由Categorical和Boosting组成。此外，CatBoost还解决了梯度偏差（Gradient Bias）以及预测偏移（Prediction shift）的问题，从而减少过拟合的发生，进而提高算法的准确性和泛化能力。

```python
def objective(trial):
    """
    定义一个 objective 函数，用于评估超参数的性能。

    参数：
        trial：一个 optuna 试验对象，用于存储超参数和结果。

    返回：
        一个 SMAPE 值，用于评估模型的性能。
    """

    # 设置超参数。
    iterations = trial.suggest_int('iterations', 100, 300)
    learning_rate = trial.suggest_float('learning_rate', 0.001, 0.3)
    depth = trial.suggest_int('depth', 1, 10)  # depth 超过 10 通常会导致过拟合
    l2_leaf_reg = trial.suggest_float('l2_leaf_reg', 0.2, 10)
    early_stopping_rounds = trial.suggest_int('early_stopping_rounds', 1, 20)

    # 创建一个 CatBoostRegressor 模型。
    model = CatBoostRegressor(
        iterations=iterations,
        learning_rate=learning_rate,
        depth=depth,
        l2_leaf_reg=l2_leaf_reg,
        early_stopping_rounds=early_stopping_rounds,
        loss_function='RMSE',
    )

    # 训练模型。
    model.fit(
        X_train, y_train,
        eval_set=[(X_train, y_train), (X_val, y_val)],
        verbose=False,
    )

    # 计算 SMAPE 值。
    smape = SMAPE(y_val, np.round(model.predict(X_val)))

    return smape

# 创建一个 optuna 试验对象。 study = optuna.create_study(direction='minimize')

# 优化超参数。study.optimize(objective, n_trials=50)

# 获取最优超参数。best_hyperparams = study.best_params
#%%
best_hyperparams = {'iterations': 282, 'learning_rate': 0.09783414836979615, 'depth': 10, 'l2_leaf_reg': 6.888098846099011, 'early_stopping_rounds': 12} # 最佳超参数
#%%
# 使用最优超参数训练模型。
model = CatBoostRegressor(**best_hyperparams)

# 训练模型。
model.fit(
    X_train, y_train,
    eval_set=[(X_train, y_train), (X_val, y_val)],
    verbose=False,
)
```

提交即可

## 总结

### 预测可视化结果：

![image](https://github.com/Sumalene/Sumalene.github.io/assets/124686994/3ae28efe-ca3a-4f8d-a157-d49b05912c3e)

其实不符合逻辑,但“的确是一种启发式方法，其假设是所有国家的销售额将在 2022 年趋于相同。这一假设并非基于任何数据或逻辑，而是基于一种试错方法，而这种方法恰好能显著提高公共评分。这种启发式方法有可能利用了公共测试集中的一些泄漏或过度拟合，但它可能无法很好地推广到私人测试集或真实世界的数据中。”

explaination from Kacper Rabczewski

---
我的成绩：
* Score: 9.11520 
* leaderboard position：202
* Public score: 8.35251

![52df560eb27412357155c43d547f40f](https://github.com/Sumalene/Sumalene.github.io/assets/124686994/0e3f4421-fad3-4978-a2fb-4a02ed5a48c2)
![d44f2eeb13b5857a701f1c5304fe07d](https://github.com/Sumalene/Sumalene.github.io/assets/124686994/22bb5ba7-1fa1-4cb5-9609-f95b84958db0)
> 从八百打到两百，应该说是数据集的问题，凹不出分，我很抱歉。
