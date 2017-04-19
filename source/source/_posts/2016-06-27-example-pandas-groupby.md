---
layout: post
title: Groupby使用小例子
date: 2016-06-27 16:51:37
tags:
	- pandas
	- groupby
	- Python
category: Python
comments: true
---

最近需要根据已有的数据计算这样一组数据：

- 股票名称
- 股票代码
- 推荐人数
- 平均分数
- 最大幅度

看到这样的需求，首先想到的是利用pandas的groupby功能。
# 一、获取数据
<!-- more -->
```
sql = "select ID, CODE, NAME, SCORE, Target from table_info"
df = pd.read_sql(sql, engine)

df.head()
Out[44]: 
                                       ID    CODE   NAME     SCORE    Target
0  {00044A0F-3D2A-49E9-B2FD-C42890B10C10}  002285    世联行  0.933333  0.020588
1  {001F206E-341A-4613-8AD5-BF9BFD3BB731}  002285    世联行  0.928571  0.147228
2  {002C6D88-A011-4E55-93F1-F13CA5B0ACD4}  000058  深 赛 格  0.800000  0.029838
3  {0036D69B-32FB-4D07-B23B-93181A573749}  300095   华伍股份  1.000000  0.071705
4  {003DEFB1-3664-4F51-B867-A0D31C63A7EE}  002588    史丹利  0.911504  0.000000
```

# 二、GroupBy处理
## 1, 获取推荐人数
```
grouped = df.groupby(['CODE','NAME'])
df_count =grouped['ID'].count()

df_count.head()
Out[48]: 
CODE    NAME
000001  平安银行    17
000006  深振业Ａ     2
000018  神州长城     3
000023  深天地Ａ     1
000026  飞亚达Ａ     6
Name: ID, dtype: int64
```


## 2, 获取平均分数
```
df_score_mean = grouped['SCORE'].mean()
df_score_mean.head()
Out[50]: 
CODE    NAME
000001  平安银行    0.899111
000006  深振业Ａ    0.943299
000018  神州长城    0.898596
000023  深天地Ａ    0.806186
000026  飞亚达Ａ    0.837766
Name: SCORE, dtype: float64

```

## 3，获取最大幅度
```
df_target_max = grouped['Target'].max()

CODE    NAME
000001  平安银行    0.096940
000006  深振业Ａ    0.054782
000018  神州长城    0.074779
000023  深天地Ａ    0.000000
000026  飞亚达Ａ    0.005254
Name: Target, dtype: float64
```
# 三、数据重组
求得了各项的值后，我们需要把这些值重组聚合起来。

利用pandas的concat函数

```
dfs = pd.concat([df_count, df_score_mean, df_target_max], axis=1, join='inner')

dfs.head()
Out[54]: 
             ID     SCORE    Target
CODE   NAME                        
000001 平安银行  17  0.899111  0.096940
000006 深振业Ａ   2  0.943299  0.054782
000018 神州长城   3  0.898596  0.074779
000023 深天地Ａ   1  0.806186  0.000000
000026 飞亚达Ａ   6  0.837766  0.005254
```

axis=1：作用在列上
join='inner'：采用内连接，取交集

由于code和name是行索引，需要把这两项变成单独的两列:

```
df_index = pd.DataFrame(dfs.index)
codesnames = df_index[0].get_values()
codes = []
names = []
for code, name in codesnames:
    codes.append(code)
    names.append(name)
dfs['code'] = codes
dfs['name'] = names

print dfs.head()


             ID     SCORE    Target    code  name
CODE   NAME                                      
000001 平安银行  17  0.899111  0.096940  000001  平安银行
000006 深振业Ａ   2  0.943299  0.054782  000006  深振业Ａ
000018 神州长城   3  0.898596  0.074779  000018  神州长城
000023 深天地Ａ   1  0.806186  0.000000  000023  深天地Ａ
000026 飞亚达Ａ   6  0.837766  0.005254  000026  飞亚达Ａ
```



