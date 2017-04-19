---
layout: post
title: pandas GroupBy使用
date: 2016-06-17 18:12:02
tags:
	- pandas
	- groupby
	- Python
	- 分组
category: Python
comments: true
---

# 一、GroupBy技术

GroupBy技术是Hadley Wickham（热门R语言包作者）创造的，用于表示分组运算，“split-apply-combine”（拆分 - 应用 - 合并）。
<!-- more -->

![](http://7xo67b.com1.z0.glb.clouddn.com/2016-06-17/groupby.png)


# 二、分组键

分组键可以有多种形式，且类型不必相同

- 列表或数组，其长度与待分组的轴一样。

- 表示DataFrame某个列名的值。

- 字典或Series，给出带分组轴上的值与分组名之间的对应关系。

- 函数，用于处理轴索引或索引中的各个标签。



# 三、分组

对于数据:
```
df = DataFrame({'key1':['a','a','b','b','a'],'key2':['one','two','one','two','one'],'data1':np.random.randn(5),'data2':np.random.randn(5)})

df

Out[7]: 

      data1     data2 key1 key2
0  0.585600  0.670239    a  one
1  1.993652  1.081585    a  two
2 -0.188506 -1.506499    b  one
3  1.016836 -0.084283    b  two
4 -1.428577 -0.464163    a  one
```

## 1，Series为分组键

按key1分组，计算data1的平均值：

```
grouped = df['data1'].groupby(df['key1'])

grouped.mean()

Out[9]: 

key1

a    0.383558
b    0.414165

Name: data1, dtype: float64

```

其索引为key1列中的唯一值。

如果我们一次传入多个数组，就会得到不同的结果。

```
means = df['data1'].groupby([df['key1'], df['key2']]).mean()

means

Out[13]: 

key1  key2
a     one    -0.421489
      two     1.993652
b     one    -0.188506
      two     1.016836
Name: data1, dtype: float64

```
    
通过两个键对数组进行分组，得到的Series具有一个层次化索引。

```
means.unstack()

Out[14]: 

key2       one       two
key1                    
a    -0.421489  1.993652
b    -0.188506  1.016836
```



## 2，数组为分组键

分组键可以是任何长度适当的数组。

```
states = np.array(['Ohio', 'Clifornia', 'California', 'Ohio', 'Ohio'])

years = np.array([2005, 2005, 2006, 2005, 2006])

df['data1'].groupby([states, years]).mean()

Out[19]: 

California  2006   -0.188506
Clifornia   2005    1.993652
Ohio        2005    0.801218
            2006   -1.428577
Name: data1, dtype: float64
```



## 3，列名为分组键

可以是字符串、数字或其他Python对象。

```
df.groupby('key1').mean()

Out[21]: 
         data1     data2
key1                    
a     0.383558  0.429221
b     0.414165 -0.795391

df.groupby(['key1','key2']).mean()

Out[22]: 

              data1     data2
key1 key2                    
a    one  -0.421489  0.103038
     two   1.993652  1.081585
b    one  -0.188506 -1.506499
     two   1.016836 -0.084283
```

> 只能对为数字的列求平均值



## 4，字典或Series为分组键

对于数据：

```
people = DataFrame(np.random.randn(5,5), columns=['a','b','c','d','e'],index=['Joe','Steve','Wes','Jim','Travis'])

people.ix[2:3,['b','c']] = np.nan

people

Out[56]: 

               a         b         c         d         e
Joe     0.561986 -0.473072  0.195847 -0.939898 -0.031220
Steve  -1.113971  1.043835 -0.759549 -0.426541 -0.175329
Wes     0.933713       NaN       NaN  0.134446  0.001153
Jim     0.262178 -1.242311 -1.482147 -0.206859  0.928791
Travis  0.714951  0.760905 -1.027728  1.124935  0.213150
```

假设已知列的分组关系，并希望根据分组计算列的总计：

```
mapping = {'a':'red', 'b':'red', 'c':'blue','d':'blue','e':'red','f':'orange'}

by_col = people.groupby(mapping, axis=1)

by_col.count()

Out[61]: 

        blue  red
Joe        2    3
Steve      2    3
Wes        1    2
Jim        2    3
Travis     2    3
```

Series也具有同样的功能，它可以被看做一个固定大小的映射。

```
map_series = Series(mapping)


map_series

Out[64]: 

a       red
b       red
c      blue
d      blue
e       red
f    orange

dtype: object


people.groupby(map_series, axis=1).count()

Out[65]: 

        blue  red
Joe        2    3
Steve      2    3
Wes        1    2
Jim        2    3
Travis     2    3

```



## 5，函数为分组键

任何被当做分组键的函数都会在各个索引值上被调用一次，其返回值就会被用作分组名称。

```
people.groupby(len).sum()

Out[66]: 

          a         b         c         d         e
3  1.757877 -1.715382 -1.286300 -1.012311  0.898724
5 -1.113971  1.043835 -0.759549 -0.426541 -0.175329
6  0.714951  0.760905 -1.027728  1.124935  0.213150

```

将函数跟数组、列表、字典、Series混合使用也不是问题，因为任何东西最终都会被转换成数组：

```
key_list = ['one']*3 + ['two']*2

people.groupby([len,key_list]).min()

Out[71]: 
              a         b         c         d         e
3 one  0.561986 -0.473072  0.195847 -0.939898 -0.031220
  two  0.262178 -1.242311 -1.482147 -0.206859  0.928791
5 one -1.113971  1.043835 -0.759549 -0.426541 -0.175329
6 two  0.714951  0.760905 -1.027728  1.124935  0.213150

```

## 6，根据索引级别分组

```
columns = pd.MultiIndex.from_arrays([['US','US', 'US', 'JP','JP'],[1,3,5,1,3]],names=['cty','tenor'])

hier_df = DataFrame(np.random.randn(4,5), columns=columns)

hier_df

Out[74]: 

cty          US                            JP          
tenor         1         3         5         1         3
0     -1.187599  0.232050  0.977682  2.480848  0.007677
1     -0.876277  0.028064  0.504568  0.229450 -0.650978
2      2.320974  0.211266  0.649322  0.222107  1.658805
3     -0.744610  0.779133  0.919815  0.333189  0.081555


hier_df.groupby(level='cty', axis=1).count()

Out[75]: 

cty  JP  US
0     2   3
1     2   3
2     2   3
3     2   3

```



## 7，对分组进行迭代

GroupBy对象支持迭代，可以产生一组二元元组。

```
for name, group in df.groupby('key1'):
    print name
    print group

    

a
      data1     data2 key1 key2
0  0.585600  0.670239    a  one
1  1.993652  1.081585    a  two
4 -1.428577 -0.464163    a  one

b
      data1     data2 key1 key2
2 -0.188506 -1.506499    b  one
3  1.016836 -0.084283    b  two

```
对于多重键的情况，元组的第一个元素将会是由键值组成的元组：

```
for name, group in df.groupby(['key1','key2']):
    print name
    print group
   
('a', 'one')
      data1     data2 key1 key2
0  0.585600  0.670239    a  one
4 -1.428577 -0.464163    a  one

('a', 'two')
      data1     data2 key1 key2
1  1.993652  1.081585    a  two

('b', 'one')
      data1     data2 key1 key2
2 -0.188506 -1.506499    b  one

('b', 'two')
      data1     data2 key1 key2
3  1.016836 -0.084283    b  two

```

还可将数组片段做出一个字典

```
pieces = dict(list(df.groupby('key1')))

pieces['b']

Out[43]: 

      data1     data2 key1 key2
2 -0.188506 -1.506499    b  one
3  1.016836 -0.084283    b  two

```

## 8，按列分组

groupby默认是在axis=0上进行分组，通过设置也可以在其他任何轴上进行分组。例如，可以根据dtype对列进行分组：

```
df.dtypes

Out[47]: 

data1    float64
data2    float64
key1      object
key2      object

dtype: object


grouped = df.groupby(df.dtypes, axis=1)

dict(list(grouped))

Out[49]: 

{dtype('float64'):       data1     data2
 0  0.585600  0.670239
 1  1.993652  1.081585
 2 -0.188506 -1.506499
 3  1.016836 -0.084283
 4 -1.428577 -0.464163, dtype('O'):   key1 key2
 0    a  one
 1    a  two
 2    b  one
 3    b  two
 4    a  one}

```



# 四、数据聚合

如果要使用自己的聚合函数，只需将其传入aggreate或agg方法即可：

```
df = DataFrame({'key1':['a','a','b','b','a'],'key2':['one','two','one','two','one'],'data1':np.random.randn(5),'data2':np.random.randn(5)})

grouped = df[['data1','data2']].groupby(df['key1'])

def peak_to_peak(arr):
    return arr.max() - arr.min()

grouped.agg(peak_to_peak)

Out[82]: 

         data1     data2
key1                    
a     0.650906  1.229130
b     0.813135  1.789667

```

表：经过优化的groupby方法

|函数名|说明|
|:----|:-----|
|count | 分组中非NA值得数量|
|sum | 非NA值得和|
|mean | 非NA值得平均值|
|median | 非NA值得算术中位数|
|std、var | 无偏（分母为n-1）标准差和方差|
|min、max | 非NA值得最小值和最大值|
|prod | 非NA值的积|
|first、last | 第一个和最后一个非NA值|


## 1，面向列的多函数应用

可以对不同列使用不同的聚合函数，或一次应用多个函数。
数据源：[tips.csv](https://raw.githubusercontent.com/wesm/pydata-book/master/ch08/tips.csv)

```
tips = pd.read_csv('https://raw.githubusercontent.com/wesm/pydata-book/master/ch08/tips.csv')

tips['tip_pct'] = tips['tip']/tips['total_bill']


tips.head()

Out[89]: 

   total_bill   tip     sex smoker  day    time  size   tip_pct
0       16.99  1.01  Female     No  Sun  Dinner     2  0.059447
1       10.34  1.66    Male     No  Sun  Dinner     3  0.160542
2       21.01  3.50    Male     No  Sun  Dinner     3  0.166587
3       23.68  3.31    Male     No  Sun  Dinner     2  0.139780
4       24.59  3.61  Female     No  Sun  Dinner     4  0.146808

```

根据sex和smoker对tips进行分组：

```
grouped = tips.groupby(['sex','smoker'])

group_pct = grouped['tip_pct']
group_pct.agg('mean')

Out[92]: 

sex     smoker
Female  No        0.156921
        Yes       0.182150
Male    No        0.160669
        Yes       0.152771

Name: tip_pct, dtype: float64

```

如果传入一组函数或函数名，得到一个DataFrame（列名默认为函数名）：

```
group_pct.agg(['mean','std',peak_to_peak])

Out[93]: 
                   mean       std  peak_to_peak
sex    smoker                                  
Female No      0.156921  0.036421      0.195876
       Yes     0.182150  0.071595      0.360233
Male   No      0.160669  0.041849      0.220186
       Yes     0.152771  0.090588      0.674707

```

可以传入由 (name, function)元组组成的列表，指定DataFrame的列名：

```
group_pct.agg([('foo','mean'),('bar',np.std)])

Out[94]: 
                    foo       bar
sex    smoker                    
Female No      0.156921  0.036421
       Yes     0.182150  0.071595
Male   No      0.160669  0.041849
       Yes     0.152771  0.090588

```

直接传入列名到函数的字典：

```
grouped.agg({'tip':np.max, 'size':'sum'})

Out[95]: 

                tip  size
sex    smoker            
Female No       5.2   140
       Yes      6.5    74
Male   No       9.0   263
       Yes     10.0   150

```

```
grouped.agg({'tip_pct':['min','max','mean','std'], 'size':'sum'})

Out[96]: 

                tip_pct                               size
                    min       max      mean       std  sum
sex    smoker                                             
Female No      0.056797  0.252672  0.156921  0.036421  140
       Yes     0.056433  0.416667  0.182150  0.071595   74
Male   No      0.071804  0.291990  0.160669  0.041849  263
       Yes     0.035638  0.710345  0.152771  0.090588  150

```

## 2，apply：一般性的“拆分 - 应用 - 合并”

定义函数，分组选出最高的5个tip_pct值。

```
def top(df, n=5, column='tip_pct'):
    return df.sort_index(by=column)[-n:]

top(tips, n=6)

Out[98]: 

     total_bill   tip     sex smoker  day    time  size   tip_pct
109       14.31  4.00  Female    Yes  Sat  Dinner     2  0.279525
183       23.17  6.50    Male    Yes  Sun  Dinner     4  0.280535
232       11.61  3.39    Male     No  Sat  Dinner     2  0.291990
67         3.07  1.00  Female    Yes  Sat  Dinner     1  0.325733
178        9.60  4.00  Female    Yes  Sun  Dinner     2  0.416667
172        7.25  5.15    Male    Yes  Sun  Dinner     2  0.710345

```

如果对smoker分组并用该函数调用apply，就会得到：

```
tips.groupby('smoker').apply(top)

Out[99]: 

            total_bill   tip     sex smoker   day    time  size   tip_pct
smoker                                                                   
No     88        24.71  5.85    Male     No  Thur   Lunch     2  0.236746
       185       20.69  5.00    Male     No   Sun  Dinner     5  0.241663
       51        10.29  2.60  Female     No   Sun  Dinner     2  0.252672
       149        7.51  2.00    Male     No  Thur   Lunch     2  0.266312
       232       11.61  3.39    Male     No   Sat  Dinner     2  0.291990
Yes    109       14.31  4.00  Female    Yes   Sat  Dinner     2  0.279525
       183       23.17  6.50    Male    Yes   Sun  Dinner     4  0.280535
       67         3.07  1.00  Female    Yes   Sat  Dinner     1  0.325733
       178        9.60  4.00  Female    Yes   Sun  Dinner     2  0.416667
       172        7.25  5.15    Male    Yes   Sun  Dinner     2  0.710345

```

如果传给apply的函数能够接受其他参数或关键字，则可以将这些内容放在函数名的后面一并传入：

```
tips.groupby('smoker').apply(top, n=1, column='total_bill')

Out[101]: 

            total_bill  tip   sex smoker  day    time  size   tip_pct
smoker                                                               
No     212       48.33    9  Male     No  Sat  Dinner     4  0.186220
Yes    170       50.81   10  Male    Yes  Sat  Dinner     3  0.196812

```


## 3，用特定于分组的值填充缺失值

对于缺失数据的清理工作，有时你会用dropna将其滤除，而有时则可能会希望用一个固定值或由数据集本身所衍生出来的值去填充NA值。这时就得用fillna工具了。

### 示例：用平均值填充NA值

```
s = Series(np.random.randn(6))
s[::2] = np.nan

s

Out[104]: 

0         NaN
1    0.596070
2         NaN
3    0.011086
4         NaN
5    0.166135

dtype: float64


s.fillna(s.mean())

Out[105]: 

0    0.257764
1    0.596070
2    0.257764
3    0.011086
4    0.257764
5    0.166135
dtype: float64

```
### 对不同的分组填充不同的值

对以下数据：

```
states = ['Ohio', 'New York', 'Vermont', 'Florida', 'Oregon', 'Nevada','California', 'Idaho']

group_key = ['East']*4 + ['West']*4

data = Series(np.random.randn(8), index = states)
data[['Vermont', 'Nevada', 'Idaho']] = np.nan


data

Out[111]: 
Ohio          0.900479
New York      0.486912
Vermont            NaN
Florida      -0.627281
Oregon       -0.580583
Nevada             NaN
California    2.033416
Idaho              NaN

dtype: float64


data.groupby(group_key).mean()

Out[112]: 

East    0.253370
West    0.726417
dtype: float64

```

用分组平均值取填充NA值

```
fill_mean = lambda g : g.fillna(g.mean())

data.groupby(group_key).apply(fill_mean)

Out[114]: 

Ohio          0.900479
New York      0.486912
Vermont       0.253370
Florida      -0.627281
Oregon       -0.580583
Nevada        0.726417
California    2.033416
Idaho         0.726417

dtype: float64

```



此外，也可以在代码中预定义各组的填充值。由于分组具有一个name属性，所以我们可以拿来用一下：

```
fill_values = {'East':0.5, 'West':-1}

fill_func = lambda g : g.fillna(fill_values[g.name])

data.groupby(group_key).apply(fill_func)

Out[118]: 

Ohio          0.900479
New York      0.486912
Vermont       0.500000
Florida      -0.627281
Oregon       -0.580583
Nevada       -1.000000
California    2.033416
Idaho        -1.000000

dtype: float64

```
















