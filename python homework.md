# Pandas 对于数据的索引与选择  
## 1.1 Pandas 里面对于数据的索引与选择 （初级）
首先是对于pandas的准备
```python
import pandas as pd
import numpy as np
```
  
官方文档给出了3种对于多维度索引的方法  
我们先随机产生一个dataframe
```python
df_rand = pd.DataFrame(np.random.randn(5, 4),
                    columns=list('ABCD'),
                    index=pd.date_range('20130101', periods=5))
df1 = df_rand
```
df1的结果如下
```python
                   A         B         C         D
2013-01-01  1.727450 -0.063898 -1.063829 -0.399406
2013-01-02 -0.375054 -0.187766 -0.003526 -0.090339
2013-01-03  0.743295  0.226209 -1.098760  1.036279
2013-01-04 -1.242288 -0.396059 -0.343701 -1.787305
2013-01-05  0.732615  1.896104 -2.192957  0.130873
```

### 1.1.1 (.loc[])用于索引 这个函数是用于对于有索引的数据进行选择  
这个函数会把需要的数据选择出来，并不改变原有的数据，比如使用
```python
df2 = df1.loc['20130102':'20130104']
#选择索引在‘20130102’和‘20130104’之间的所有数据

df2 = df1.loc['20130102','20130103']
#选择索引为‘20130102’和‘20130103’的数据

```
不仅如此，在.loc[]中还可以是一个返回值为已有索引的可调用的函数（这个函数可以有多个参数，但是返回值只能是其中一个索引而不是多个） 例子如下
```python
def test_1(test_number):
    if test_number == 1:
        return '20130101'
    else:
        return '20130102'

print(df1.loc[test_1(1)])

#结果如下
A   -0.917945
B   -0.493247
C    1.377406
D    2.060027
```  
还可以进行多维选择  
```python
print(df1.loc['20130102':'20130104', 'A':'C'])

#结果如下
                   A         B         C
2013-01-02  1.220908  0.716535 -0.128297
2013-01-03 -0.491150 -0.212621  1.182051
2013-01-04 -0.667444  2.432083 -0.884137
```

除了选择dataframe类型的数据，同样可以操作series类型的，方法与上相同  
此外，还可以通过.loc[]进行数据的判断,赋值和添加  
```python
series_1 = pd.Series(np.array([1, 2, 3]), index=list('abc'))
print(series_1)
series_1.loc['b':] = 0
print(series_1)
print(series_1.loc['b':] > 2)
#结果如下
a    1
b    2
c    3
dtype: int32
a    1
b    0
c    0
dtype: int32
b    False
c    False
dtype: bool
```
### 1.1.2 (.iloc[])对于数据的索引 
这个函数是通过位置而非索引进行索引与选择的  
与.loc[]相似，函数的参数有一个特定的顺序：选择先按照index，之后按照columns的名字进行选择，同样是左边数字不包括在内。
```python
print(df1.iloc[1:3, 2:3])

#结果如下
                   C
2013-01-02  0.274570
2013-01-03 -1.608922
```
### 1.1.3 [] 的使用
这个取索引的方法在python中十分普遍，在此不再赘述。  

### 1.1.4 三种方法在时间复杂度方面的对比
下面是对于这三种方法的时间复杂度测试
```python
import time
times = 0
def time_calculate(index_type, df_test):
    if index_type == 0:
        time_start = time.time()
        for times in range(1, 10000):
            df_test.loc['20130102':'20130103']
        time_end = time.time()
        time_use = time_end - time_start
        print(".loc[]用时：",time_use)
    elif index_type == 1:
        time_start = time.time()
        for times in range(1, 10000):
            df_test.iloc[1:2]
        time_end = time.time()
        time_use = time_end - time_start
        print(".iloc[]用时：",time_use)
    else:
        time_start = time.time()
        for times in range(1, 10000):
            df_test[1:2]
        time_end = time.time()
        time_use = time_end - time_start
        print("[]用时：",time_use)

time_calculate(0,df1)
time_calculate(1,df1)
time_calculate(2,df1)

#结果如下
.loc[]用时： 6.189121246337891
.iloc[]用时： 1.4947333335876465
[]用时： 1.4722979068756104
```
我们可以明显发现.loc[]函数所需时间更长，二后两者区别不大，所以在处理大量的数据时，尽量不要使用索引来进行索引 ，更多采用位置尽行索引 



### 1.1.5 sample 函数  
此函数用于从一个series或者dataframe中获取随机的数据  
这个函数有以下几个很有用的参数  
首先来看一下这个函数  
```python
def sample(n=None, frac=None, replace=False, weights=None, random_state=None, axis=None)
```
参数n是指要从中选择出来的样本个数  
```python
print(df1.sample(n=2))
#结果如下
                   A         B         C         D
2013-01-01  1.343622 -0.243604  1.189220 -0.405317
2013-01-04  0.671955  0.484274  0.875337 -0.115321
```
另外一个参数是axis
```python
# axis = 1是指纵向， axis = 0是和默认一样，横向抽样
print(df1.sample(n=2, axis=1))
#结果如下
                   C         D
2013-01-01 -2.191866 -0.135250
2013-01-02 -1.070149 -1.273672
2013-01-04 -0.109600  0.682061
2013-01-05 -0.578205 -0.006628
```
replace参数是指是否重复抽样，如果想要重复抽样，把默认参数改为True即可  

frac参数是指的是选取其中的数据的比例，此时，replace参数默认为True
```python
print(df1.sample(frac=0.2))
# 0.2 * 5 = 1 因此会抽取一个样本，如果不是整数，会进行四舍五入计算抽样数量
#结果如下
                   A         B         C         D
2013-01-03  0.028262 -0.319101 -0.646526  1.554117
```
weights参数非常有用，可以选择样本总体中各个样本的权重  
先建一个list来存储权重即可
```python
list_weights = [0.1, 0.1, 0.2, 0.2, 0.4]
print(df1.sample(n=2, weights=list_weights))
#结果如下
                   A         B         C         D
2013-01-01  0.181466 -0.582357  0.685201  0.837260
2013-01-05 -0.302139 -0.597710  0.315432 -1.307208
```
random_state和numpy中的seed是一样的，用于改变随机取样的方法  

### 1.1.6 reindex 函数
当你的数据中存在有丢失数据时，可以通过此方法进行，据官方文档，在Pandas 0.21.0之后就不允许在这种情况下使用.loc[]  
```python
print(series_1.loc[['a', 'b', 'c', 'd']])

KeyError: 'Passing list-likes to .loc or [] with any missing labels is no longer supported
# 使用reindex
print(series_1.reindex(['a', 'b', 'c', 'd']))

a    1.0
b    2.0
c    3.0
d    NaN
dtype: float64
```
### 1.1.7 快速索引与选择
很多情况下我们需要快速进行选择，因此我们可以通过  
.at[] 和 .iat[] .at[]与.loc[]相似 .iat[]与.iloc[]相似  
这两个函数体现的是坐标的意思，比如是一个二维的表，可以用两个参数来确定位置，然后返回数据
```python
print(series_1.iat[2])
print(series_1.at['a'])
print(df1)
print(df1.iat[0, 1])
#结果如下
3
1
                   A         B         C         D
2013-01-01  0.849127 -1.053560  0.240735  0.470015
2013-01-02 -0.274833  0.990401  0.047524  1.831534
2013-01-03 -0.599481  2.258076 -0.308622  0.464663
2013-01-04  0.053717  0.771135  0.738720 -0.447455
2013-01-05 -0.298861 -1.099683 -0.191119 -1.496427
-1.0535597235755005
```

### 1.1.8 Boolean方式进行的索引与选择
格式如下  
值得注意的是在pandas里面，与或的符号是“&”，“|”
```python
series_2 = pd.Series(range(-3, 4))
print(series_2[(series_2 < 1) | (series_2 > 3)])
#结果如下
0   -3
1   -2
2   -1
3    0
dtype: int64
```
### isin 函数判断是否在数据中
```python
series_2.isin([2, 3])

#结果如下
0    False
1    False
2    False
3    False
4    False
5     True
6     True
dtype: bool
```
 可以同时选择使用.all()或者.any()来快速选择满足条件的所有数据  
 例如
 ```python
df3 = pd.DataFrame({'vals': [1, 2, 3, 4], 'ids': ['a', 'b', 'c', 'd'], 'ids2':['a', 'n', 'c', 'n']})
values = {'ids': ['a', 'b'], 'vals': [1, 3], 'ids2': ['a', 'c']}
print(df3.isin(values))
row_info = df3.isin(values).any(1)
#这里1是指行
print(df3.any(1))

# 结果如下
    vals    ids   ids2
0   True   True   True
1  False   True  False
2   True  False   True
3  False  False  False
   vals ids ids2
0     1   a    a
1     2   b    n
2     3   c    c

 ```
 
 ### 1.1.9 where 函数







## Pandas 对于数据的索引与选择 （高级）
