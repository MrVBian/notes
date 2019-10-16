# numpy

------

#### txt数据读入

```python
import numpy
# skip_header=1略过标题
world_alcohol = numpy.genfromtxt('world_alcohol.txt', delimiter = ',', dtype = str)
print(world_alcohol)
```

------

#### 矩阵声明及属性

```python
# array所有元素类型相同
vector = numpy.array([5, 10, 15, 20])
matrix = numpy.array([[5, 10, 15], [20, 25, 30], [35, 40, 45]])
# 数据大小
print(matrix.shape)
# 据类型
print(matrix.dtype)
```
------

```python
#读取txt内容
# skip_header=1略过标题
world_alcohol = numpy.genfromtxt('world_alcohol.txt', delimiter = ',', dtype = str)
res = world_alcohol[1, 4]
# 切片
res = world_alcohol[1: 4]
```
------

#### 矩阵切片

```python
matrix = numpty.array([
                    [5, 10, 15],
                    [20, 25, 30],
                   [35, 10, 15]
                    ])
# 切片，所有行的下标为1的列
print(matrix[:, 1])
# 切片，所有行的下标为0 ~ 1的列 
print(matrix[:, 0:2])
```

------

#### 矩阵基本操作 1

```python
matrix = numpty.array([
                    [5, 10, 15],
                    [20, 25, 30],
                    [35, 10, 15]
                    ])
# 逐个较是否等于
matrix == 25

# 找出下标1列是否含25
matri[:, 1] == 25

# 既等于10 又 等于5的数
(matrix == 10) & (matrix == 5)

# 等于10 或 等于5的数
(matrix == 10) | (matrix == 5)

# 查看数据类型
matrix.dtype

# 转换为float类型
matrix.astype(float)

# 最大值
matrix.max()

# 以行为单位求和
matrix.sum(axis = 1)

# 以列为单位求和
matrix.sum(axis = 0)

```

------

#### 矩阵基本操作 1

```python
# 生成长度为15的矩阵，值从[0,14]
a = np.arange(15)

# 生成以[10, 30)，间隔为5的矩阵
np.arange( 10, 30, 5 )

# 将矩阵结构改成3*5 shape直接改变原矩阵，reshape不改变原矩阵
a = np.arange(15).reshape(3, 5)
a.shape = (6, 2)

# 矩阵维度（axes）
a.ndim

# 矩阵类型名
a.dtype.name

# 矩阵大小
a.size

# 生成3*5的零矩阵
np.zeros( (3, 4) )

# 生成2*3*4的1矩阵，变量类型为int32
np.ones( (2, 3, 4), dtype = np.int32 )

# 生成2*3的0-1随机数
np.random.random( (2, 3) )

# 等差数列 
from numpy import pi
np.linspace( 0, 2*pi, 100 )
# [0, 100] 9个数的等差
np.linspace( 0, 100, 9 )

# 维度相同的矩阵 作基本运算，对应位置计算
# 点乘运算
a.dot(b)
```

------

#### 矩阵基本运算 3

```python
b = np.arange(3)

# e的多少次方
np.exp(b)

# 开根
np.sqrt(b)

# 返回floor
a = np.floor( 10*np.random.random( (3, 4) ) )

# 将矩阵拉成一维向量
a.ravel()

# 将矩阵改为6 * 2 shape直接改变原矩阵，reshape不改变原矩阵
a.shape = (6, 2)
a.reshape( (6, 2) )

# 矩阵转置
a.T

# 自动计算缺省维度，第一维度为3，第二维度自动计算
a.reshape(3, -1)
```

------

#### 矩阵拼接

```python
a = np.floor(10 * np.random.random(2, 2))
b = np.floor(10 * np.random.random(2, 2))

# 横向拼接
np.hstack( (a, b) )
# 纵向拼接
np.vstack( (a, b) )

# 横向切分
# 切成三份
np.hsplit(a, 3)
# 自定义切分，在第3、4列切两下
np.hsplit(a, (3, 4))
# 横向切分
np.vsplit(a, 3)
np.vsplit(a, (3, 4))
```

------

#### 复制

```python
# a、b完全相关
a = np.arange(12)
b = a
print( b is a ) # 结果为true
id(a) == id(b)  # 结果为true

# 虽然a、c的id不同，但是值共用，修改c，a的值也变化
c = a.view()
print( c is a ) # 结果为false

# a、d完全无关
d = a.copy()
print( d is a ) # 结果为false
```

------

#### 找出每列最大数

```python
data = np.sin(np.arange(20)).reshape(5,4)
print(data)
# 每列最大数位置
ind = data.argmax(axis = 0)
print(ind)
data_max = data[ind, range(data.shape[1])] # range(data.shape[1]) (0,4)
print(data_max)
```

------

#### 扩展

```python
a = np.arange(0, 40, 10)
# 行扩展四倍，列扩展三倍
b = np.tile(a, (4, 3))
```

------

#### 排序

```python
a = np.array( [[4, 3, 5], [1, 2, 1]] )
# 对行排序，默认从大到小
b = np.sort(a, axis = 1)

# 返回排序后的下标，默认从小到大排列
a = np.array( [4, 3, 1, 2] )
c = np.argsort(a)

```




