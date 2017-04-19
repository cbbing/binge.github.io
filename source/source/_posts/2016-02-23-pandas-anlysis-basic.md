---
layout: post
title: Pandas数据分析基础
date: 2016-02-23 10:27:49
tags:
	- pandas
	- DataFrame
	- 数据分析
	- Python

category: Python
comments: true
---

使用pandas，首先导入包：

```
from pandas import Series, DataFrame
import pandas as pd

```


# 一、创建Series，DataFrame

## 1，创建Series

### a，通过列表创建

```
obj = Series([4, 7, -5, 3]) 
obj2 = Series([4, 7, -5, 3], index=['d','b','a','c']) #指定索引

```
<!-- more -->
### b，通过字典创建Series

```
sdata = {'Ohio':35000, 'Texas':7100, 'Oregon':1600,'Utah':500}
obj3 = Series(sdata)

```


### c，通过字典 + 索引

```
states = ['California', 'Ohio', 'Oregon', 'Texas']
obj4 = Series(sdata, index=states)
```

> 指定索引时，跟states索引匹配的那3个值会被找出并放到相应的位置，‘California'对应的sdata值找不到，其结果为NaN。



## 2，创建DataFrame

### a，词典生成

```
data = {'state':['Ohio', 'Ohio', 'Ohio', 'Nevada','Nevada'],
        'year':[2000, 2001, 2002, 2011, 2002],
        'pop':[1.5, 1.7, 3.6, 2.4, 2.9]}
frame = DataFrame(data)

frame2 = DataFrame(data, columns=['year', 'state', 'pop']) #指定列
frame3 = DataFrame(data, columns=['year', 'state', 'pop']，
         index=['one', 'two', 'three', 'four', 'five']) #指定列和索引
```

### b，列表生成

```
>>> errors = [('c',1,'right'), ('b', 2,'wrong')]
>>> df = pd.DataFrame(errors)
>>> df
   0  1      2
0  c  1  right
1  b  2  wrong

>>> df = pd.DataFrame(errors, columns=['name', 'count', 'result'])  #指定列名
>>> df
  name  count result
0    c      1  right
1    b      2  wrong
```

### c, 嵌套词典（也就是词典的词典）

```
pop = {'Nevada':{2001:2.4, 2002:2.9},
       'Ohio':{2000:1.5, 2001:1.7, 2002:3.6}}
frame4 = DataFrame(pop)
Out[138]:
      Nevada  Ohio
2000     NaN   1.5
2001     2.4   1.7
2002     2.9   3.6
```

### d，Series组合

#### 按行生成DataFrame

```
In [4]: a = pd.Series([1,2,3]) 
In [5]: b = pd.Series([2,3,4])
In [6]: c = pd.DataFrame([a,b]) 
In [7]: c
Out[7]:
   0  1  2
0  1  2  3
1  2  3  4
```

#### 按列生成DataFrame

```
In [8]: c = pd.DataFrame({'a':a,'b':b})
In [9]: c
Out[9]:
   a  b
0  1  2
1  2  3
2  3  4
```


# 二，选取

对于一组数据DataFrame：

```
data = DataFrame(np.arange(16).reshape((4,4)),index=['Ohio', 'Colorado','Utah','New York'],columns=['one','two','three','four'])
>>> data
          one  two  three  four
Ohio        0    1      2     3
Colorado    4    5      6     7
Utah        8    9     10    11
New York   12   13     14    15
```

## 1，选取列，返回一个Series

```
>>> data['two']
Ohio         1
Colorado     5
Utah         9
New York    13
Name: two, dtype: int64
```

## 2，选取行，返回一个Series

```
>>> data.ix['Ohio']
one      0
two      1
three    2
four     3
Name: Ohio, dtype: int64
```

## 3， 选取行和列, 可以是行名，列名，或列的序号

```
>>> data.ix['Ohio', ['two','three']]
two      1
three    2
Name: Ohio, dtype: int64
```

```
>>> data.ix[data.three > 3, :3]
          one  two  three
Colorado    4    5      6
Utah        8    9     10
New York   12   13     14
```

# 三、遍历与汇总

## 1，按行遍历

```
for ix, row in df.iterrows():
```



## 2，按列遍历

```
for ix, col in df.iteritems():
```

## 3，汇总

```
In[95]: frame = DataFrame({'b':[4, 7, -3, 2], 'a':[0, 1, 0, 1]})
In[99]: frame.sum()
Out[99]: 
a     2
b    10
dtype: int64
```

# 四、排序

## 1，对索引排序

对轴索引排序

Series用sort_index()按索引排序，sort()按值排序；

DataFrame用sort_index()和sort()是一样的。

```
In[73]: obj = Series(range(4), index=['d','a','b','c'])
In[74]: obj.sort_index()  
Out[74]: 
a    1
b    2
c    3
d    0
dtype: int64

In[78]: frame = DataFrame(np.arange(8).reshape((2,4)),index=['three', 'one'],columns=['d','a','b','c'])
In[79]: frame
Out[79]: 
       d  a  b  c
three  0  1  2  3
one    4  5  6  7

In[86]: frame.sort_index()
Out[86]: 
       d  a  b  c
one    4  5  6  7
three  0  1  2  3

In[87]: frame.sort()
Out[87]: 
       d  a  b  c
one    4  5  6  7
three  0  1  2  3
```

## 2，按行排序

```
In[89]: frame.sort_index(axis=1, ascending=False)
Out[89]: 
       d  c  b  a
three  0  3  2  1
one    4  7  6  5
```

## 3，按列排序（只针对Series）

```
In[90]: obj.sort()
In[91]: obj
Out[91]: 
d    0
a    1
b    2
c    3
dtype: int64
```

## 4，按值排序

Series:

```
In[92]: obj = Series([4, 7, -3, 2])
In[94]: obj.order()
Out[94]: 
2   -3
3    2
0    4
1    7
dtype: int64
```

DataFrame:

```
In[95]: frame = DataFrame({'b':[4, 7, -3, 2], 'a':[0, 1, 0, 1]})
In[97]: frame.sort_index(by='b')
Out[97]: 
   a  b
2  0 -3
3  1  2
0  0  4
1  1  7
```



# 五、删除

## 1，删除指定轴上的项

即删除 Series 的元素或 DataFrame 的某一行（列）的意思，通过对象的 .drop(labels, axis=0) 方法：

删除Series的一个元素:

```
In[11]: ser = Series([4.5,7.2,-5.3,3.6], index=['d','b','a','c'])
In[13]: ser.drop('c')
Out[13]: 
d    4.5
b    7.2
a   -5.3
dtype: float64
```

删除DataFrame的行或列：

```
In[17]: df = DataFrame(np.arange(9).reshape(3,3), index=['a','c','d'], columns=['oh','te','ca'])
In[18]: df
Out[18]: 
   oh  te  ca
a   0   1   2
c   3   4   5
d   6   7   8

In[19]: df.drop('a')
Out[19]: 
   oh  te  ca
c   3   4   5
d   6   7   8

In[20]: df.drop(['oh','te'],axis=1)
Out[20]: 
   ca
a   2
c   5
d   8
```

.drop() 返回的是一个新对象，元对象不会被改变。



# 六、DataFrame连接

## 1，算术运算（+，-，*，/）

是df中对应位置的元素的算术运算

```
In[5]: df1 = DataFrame(np.arange(12.).reshape((3,4)),columns=list('abcd'))

In[6]: df2 = DataFrame(np.arange(20.).reshape((4,5)),columns=list('abcde'))

In[9]: df1+df2
Out[9]: 
    a   b   c   d   e
0   0   2   4   6 NaN
1   9  11  13  15 NaN
2  18  20  22  24 NaN
3 NaN NaN NaN NaN NaN
```

传入填充值

```
In[11]: df1.add(df2, fill_value=0)
Out[11]: 
    a   b   c   d   e
0   0   2   4   6   4
1   9  11  13  15   9
2  18  20  22  24  14
3  15  16  17  18  19
```

## 2，pandas.merge

pandas.merge可根据一个或多个键将不同DataFrame中的行连接起来。

默认情况下，merge做的是“inner”连接，结果中的键是交集，其它方式还有“left”，“right”，“outer”。“outer”外连接求取的是键的并集，组合了左连接和右连接。

### 内连接

```
In[14]: df1 = DataFrame({'key':['b','b','a','c','a','a','b'],'data1':range(7)})

In[15]: df2 = DataFrame({'key':['a','b','d'],'data2':range(3)})

In[18]: pd.merge(df1, df2)  #或显式: pd.merge(df1, df2, on='key')
Out[18]: 
   data1 key  data2
0      0   b      1
1      1   b      1
2      6   b      1
3      2   a      0
4      4   a      0
5      5   a      0
```

### 外连接

```
In[19]: pd.merge(df1, df2, how='outer')
Out[19]: 
   data1 key  data2
0      0   b      1
1      1   b      1
2      6   b      1
3      2   a      0
4      4   a      0
5      5   a      0
6      3   c    NaN
7    NaN   d      2
```



### 轴向连接

这种数据合并运算被称为连接（concatenation）、绑定（binding）或堆叠（stacking）。

对于Series

```
In[23]: s1 = Series([0, 1], index=['a','b'])
In[24]: s2 = Series([2, 3, 4], index=['c','d','e'])
In[25]: s3 = Series([5, 6], index=['f','g'])

In[26]: pd.concat([s1,s2,s3])
Out[26]: 
a    0
b    1
c    2
d    3
e    4
f    5
g    6
dtype: int64
```
默认情况下，concat是在axis=0（行）上工作的，最终产生一个新的Series。如果传入axis=1（列），则变成一个DataFrame。

```
In[27]: pd.concat([s1,s2,s3], axis=1)
Out[27]: 
    0   1   2
a   0 NaN NaN
b   1 NaN NaN
c NaN   2 NaN
d NaN   3 NaN
e NaN   4 NaN
f NaN NaN   5
g NaN NaN   6
```

DataFrame连接

```
dfs = []
for classify in classify_finance + classify_other:
    sql = "select classify, tags from {} where classify='{}' length(tags)>0 limit 1000".format(mysql_table_sina_news_all, classify)
    df = pd.read_sql(sql,engine)
    dfs.append(df)
df_all = pd.concat(dfs, ignore_index=True)
```


# 七、数据转换

数据过滤、清理以及其他的转换工作。

## 1，移除重复数据（去重）

### duplicated()

DataFrame的duplicated方法返回一个布尔型Series，表示各行是否是重复行：

```
In[12]: df = DataFrame({'k1':['one']*3 + ['two']*4, 'k2':[1,1,2,3,3,4,4]})

In[13]: df
Out[13]: 
    k1  k2
0  one   1
1  one   1
2  one   2
3  two   3
4  two   3
5  two   4
6  two   4

In[14]: df.duplicated()
Out[14]: 
0    False
1     True
2    False
3    False
4     True
5    False
6     True
dtype: bool

```



### drop_duplicates()

```
In[15]: df.drop_duplicates()
Out[15]: 
    k1  k2
0  one   1
2  one   2
3  two   3
5  two   4
```



## 2，利用函数或映射进行数据转换

对于数据:

```
In[16]: df = DataFrame({'food':['bacon','pulled pork','bacon','Pastraml','corned beef', 'Bacon', 'pastraml','honey ham','nova lox'],'ounces':[4,3,12,6,7.5,8,3,5,6]})

In[17]: df
Out[17]: 
          food  ounces
0        bacon     4.0
1  pulled pork     3.0
2        bacon    12.0
3     Pastraml     6.0
4  corned beef     7.5
5        Bacon     8.0
6     pastraml     3.0
7    honey ham     5.0
8     nova lox     6.0
```

增加一列表示该肉类食物来源的动物类型，先编写一个肉类到动物的映射：

```
In[18]: meat_to_animal = {'bacon':'pig',
    'pulled pork':'pig',
    'pastraml':'cow',
    'corned beef':'cow',
    'honey ham':'pig',
    'nova lox':'salmon'}
```

### map

Series的map方法可以接受一个函数或含有映射关系的字典型对象。

```
In[20]: df['animal'] = df['food'].map(str.lower).map(meat_to_animal)
In[21]: df
Out[21]: 
          food  ounces  animal
0        bacon     4.0     pig
1  pulled pork     3.0     pig
2        bacon    12.0     pig
3     Pastraml     6.0     cow
4  corned beef     7.5     cow
5        Bacon     8.0     pig
6     pastraml     3.0     cow
7    honey ham     5.0     pig
8     nova lox     6.0  salmon
```



也可传入一个函数，一次性处理：

```
In[22]: df['food'].map(lambda x : meat_to_animal[x.lower()])
Out[22]: 
0       pig
1       pig
2       pig
3       cow
4       cow
5       pig
6       cow
7       pig
8    salmon
Name: food, dtype: object
```
### apply 和 applymap

对于DataFrame：

```
In[21]: df = DataFrame(np.random.randn(4,3), columns=list('bde'),index=['Utah','Ohio','Texas','Oregon'])

In[22]: df
Out[22]: 
               b         d         e
Utah    1.654850  0.594738 -1.969539
Ohio    2.178748  1.127218  0.451690
Texas   1.209098 -0.604432 -1.178433
Oregon  0.286382  0.042102 -0.345722
```

apply将函数应用到由各列或行所形成的一维数组上。

**作用到列：**

```
In[24]: f = lambda x : x.max() - x.min()
In[25]: df.apply(f)
Out[25]: 
b    1.892366
d    1.731650
e    2.421229
dtype: float64
```

**作用到行/轴：**

```
In[26]: df.apply(f, axis=1)
Out[26]: 
Utah      3.624390
Ohio      1.727058
Texas     2.387531
Oregon    0.632104
dtype: float64
```

**作用到每个元素：**

```
In[70]: frame = DataFrame(np.random.randn(4,3), columns=list('bde'),index=['Utah','Ohio','Texas','Oregon'])

In[72]: frame.applymap(lambda x : '%.2f' % x)
Out[72]: 
            b      d      e
Utah     1.19   1.56  -1.13
Ohio     0.10  -1.03  -0.04
Texas   -0.22   0.77  -0.73
Oregon   0.22  -2.06  -1.25
```








### numpy的ufuncs

Numpy的ufuncs（元素级数组方法）也可用于操作pandas对象。



取绝对值操作

```
In[23]: np.abs(df)
Out[23]: 
               b         d         e
Utah    1.654850  0.594738  1.969539
Ohio    2.178748  1.127218  0.451690
Texas   1.209098  0.604432  1.178433
```






## 3，替换值

替换的几种形式

```
In[23]: se = Series([1, -999, 2, -999, -1000, 3])
In[24]: se.replace(-999, np.nan)
Out[24]: 
0       1
1     NaN
2       2
3     NaN
4   -1000
5       3
dtype: float64

In[25]: se.replace([-999, -1000], np.nan)
Out[25]: 
0     1
1   NaN
2     2
3   NaN
4   NaN
5     3
dtype: float64

In[26]: se.replace([-999, -1000], [np.nan, 0])
Out[26]: 
0     1
1   NaN
2     2
3   NaN
4     0
5     3
dtype: float64


# 字典
In[27]: se.replace({-999:np.nan, -1000:0})
Out[27]: 
0     1
1   NaN
2     2
3   NaN
4     0
5     3
dtype: float64
```

## 4，重命名轴索引、列名

对于数据:

```
In[28]: df = DataFrame(np.arange(12).reshape((3,4)), index = ['Ohio', 'Colorado', 'New York'], columns=['one','two','three', 'four'])

In[29]: df
Out[29]: 
          one  two  three  four
Ohio        0    1      2     3
Colorado    4    5      6     7
New York    8    9     10    11
```

就地修改轴索引

```
In[30]: df.index = df.index.map(str.upper)
In[31]: df
Out[31]: 
          one  two  three  four
OHIO        0    1      2     3
COLORADO    4    5      6     7
NEW YORK    8    9     10    11
```
如果要创建数据集的转换版（而不是修改原始数据），比较实用的方法是rename：

```
In[32]: df.rename(index=str.title, columns=str.upper)
Out[32]: 
          ONE  TWO  THREE  FOUR
Ohio        0    1      2     3
Colorado    4    5      6     7
New York    8    9     10    11
```

特别说明一下，rename可以结合字典型对象实现对部分轴标签的更新：

```
In[33]: df.rename(index={'OHIO':'INDIANA'}, columns={'three':'peekaboo'})
Out[33]: 
          one  two  peekaboo  four
INDIANA     0    1         2     3
COLORADO    4    5         6     7
NEW YORK    8    9        10    11
```
如果希望就地修改某个数据集，传入inplace=True即可：

```
In[34]: _ = df.rename(index={'OHIO':'INDIANA'}, inplace=True)
In[35]: df
Out[35]: 
          one  two  three  four
INDIANA     0    1      2     3
COLORADO    4    5      6     7
NEW YORK    8    9     10    11
```



## 5，离散化和面元划分

### pd.cut

为了便于分析，连续数据常常离散化或拆分为“面元”（bin）。比如：

```
In [106]: ages = [20, 22,25,27,21,23,37,31,61,45,41,32]
```

需要将其划分为“18到25”,  “26到35”，“36到60”以及“60以上”几个面元。要实现该功能，需要使用pandas的cut函数。

```
n[37]: bins = [18, 25, 35, 60, 100]
In[38]: cats = pd.cut(ages, bins)
In[39]: cats
Out[39]: 
[(18, 25], (18, 25], (18, 25], (25, 35], (18, 25], ..., (25, 35], (60, 100], (35, 60], (35, 60], (25, 35]]
Length: 12
Categories (4, object): [(18, 25] < (25, 35] < (35, 60] < (60, 100]]
```

可以通过right=False指定哪端是开区间或闭区间。

```
In[41]: cats = pd.cut(ages, bins, right=False)
In[42]: cats
Out[42]: 
[[18, 25), [18, 25), [25, 35), [25, 35), [18, 25), ..., [25, 35), [60, 100), [35, 60), [35, 60), [25, 35)]

Length: 12

Categories (4, object): [[18, 25) < [25, 35) < [35, 60) < [60, 100)]

```

也可以指定面元的名称：

```
In[43]: group_name = ['Youth', 'YoungAdult', 'MiddleAged', 'Senior']

In[45]: cats = pd.cut(ages, bins, labels=group_name)
In[47]: cats
Out[47]: 
[Youth, Youth, Youth, YoungAdult, Youth, ..., YoungAdult, Senior, MiddleAged, MiddleAged, YoungAdult]

Length: 12

Categories (4, object): [Youth < YoungAdult < MiddleAged < Senior]


In[46]: pd.value_counts(cats)
Out[46]: 
Youth         5
MiddleAged    3
YoungAdult    3
Senior        1
dtype: int64
```

### pd.qcut

qcut是一个非常类似cut的函数，它可以根据样本分位数对数据进行面元划分，根据数据的分布情况，cut可能无法使各个面元中含有相同数量的数据点，而qcut由于使用的是样本分位数，可以得到大小基本相等的面元。

```
In[48]: data = np.random.randn(1000)
In[49]: cats = pd.qcut(data, 4)
In[50]: cats
Out[50]: 
[(0.577, 3.564], (-0.729, -0.0341], (-0.729, -0.0341], (0.577, 3.564], (0.577, 3.564], ..., [-3.0316, -0.729], [-3.0316, -0.729], (-0.0341, 0.577], [-3.0316, -0.729], (-0.0341, 0.577]]

Length: 1000

Categories (4, object): [[-3.0316, -0.729] < (-0.729, -0.0341] < (-0.0341, 0.577] < (0.577, 3.564]]


In[51]: pd.value_counts(cats)
Out[51]: 
(0.577, 3.564]       250
(-0.0341, 0.577]     250
(-0.729, -0.0341]    250
[-3.0316, -0.729]    250
dtype: int64
```


## 6，检测和过滤异常值

异常值（oulier）的过滤或变换运算在很大程度上其实就是数组运算。

对于数据：

```
In[52]: np.random.seed(12345)
In[53]: data = DataFrame(np.random.randn(1000,4))
In[54]: data.describe()
Out[54]: 
                 0            1            2            3
count  1000.000000  1000.000000  1000.000000  1000.000000
mean     -0.067684     0.067924     0.025598    -0.002298
std       0.998035     0.992106     1.006835     0.996794
min      -3.428254    -3.548824    -3.184377    -3.745356
25%      -0.774890    -0.591841    -0.641675    -0.644144
50%      -0.116401     0.101143     0.002073    -0.013611
75%       0.616366     0.780282     0.680391     0.654328
max       3.366626     2.653656     3.260383     3.927528
```
找出某列绝对值大于3的值

```
In[55]: col = data[3]
In[56]: col[np.abs(col) > 3]
Out[56]: 
97     3.927528
305   -3.399312
400   -3.745356
Name: 3, dtype: float64
```

要选出全部含有“超过3或-3的值”的行，可以利用布尔型DataFrame以及any方法：

```
In[60]: data[(np.abs(data)>3).any(1)]
Out[60]: 
            0         1         2         3
5   -0.539741  0.476985  3.248944 -1.021228
97  -0.774363  0.552936  0.106061  3.927528
102 -0.655054 -0.565230  3.176873  0.959533
305 -2.315555  0.457246 -0.025907 -3.399312
324  0.050188  1.951312  3.260383  0.963301
400  0.146326  0.508391 -0.196713 -3.745356
499 -0.293333 -0.242459 -3.056990  1.918403
523 -3.428254 -0.296336 -0.439938 -0.867165
586  0.275144  1.179227 -3.184377  1.369891
808 -0.362528 -3.548824  1.553205 -2.186301
900  3.366626 -2.372214  0.851010  1.332846
```

根据这些条件，可以轻松对值进行设置，下面代码将值限制在区间-3到3以内：

```
In[62]: data[np.abs(data)>3] = np.sign(data)*3
In[63]: data.describe()
Out[63]: 
                 0            1            2            3
count  1000.000000  1000.000000  1000.000000  1000.000000
mean     -0.067623     0.068473     0.025153    -0.002081
std       0.995485     0.990253     1.003977     0.989736
min      -3.000000    -3.000000    -3.000000    -3.000000
25%      -0.774890    -0.591841    -0.641675    -0.644144
50%      -0.116401     0.101143     0.002073    -0.013611
75%       0.616366     0.780282     0.680391     0.654328
max       3.000000     2.653656     3.000000     3.000000
```


















