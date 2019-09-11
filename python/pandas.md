# pandas

---

#### 数据读取及属性

```python
import pandas
food_info = pandas.read_csv("food_info.csv")
print(type(food_info))
print(food_info.dtypes)
```

------

#### 基本操作

```python
food_info.head()
food_info.head(3)
food_info.tail()
food_info.tail(3)
food_info.columns() 列名
food_info.shape()   数据长宽大小
```

------

#### 下标与分片

```python
food_info.loc[0]    # 第一行数据
food_info.loc[3:6]
# Return a DataFrame containing the rows at indexes 2, 5, and 10。
# Method 1
two_five_ten = [2, 5, 10]
food_info.loc[two_five_ten]
# Method 2
food_info.loc[[2, 5, 10]]
```

------

#### 类型

> * object - For string values
> * int - For integer values
> * float - For float values
> * datetime - For time values
> * bool - For Boolean values
> * print(food_info.dtypes)

------

#### 选取某列

```python
# method 1
food_info['NDB']
# method 2
col_name = 'NDB'
food_info[col_name]
```

------

#### 转化为数组

```python
col_names = food_info.columns.tolist()
```

#### 把单位为g的列选择出来

``` python
gram_columns = []
for c in col_names:
    if c.endswith("(g)"):
        gram_columns.append(c)
gram_df = food_info[gram_columns]
print( gram_df.head(3) )
```    

#### g单位换算成kg单位

```python
res = food_info["Iron_(mg)"] / 1000
print(res)
```

------

####  基本运算 1

```python
# 两个相同维度的列 乘法 是对应维度参数相乘
water_energy = food_info['Water_(g)'] * food_info['Energ_Kcal']
iron_grams = food_info['Iron_(mg)'] / 1000
print(food_info.shape)
# 增加一个列Iron_(g)
food_info['Iron_(g)'] = iron_grams
print(food_info.shape)
```

#### 基本运算 2

```python
# 每行数据*2
weighted_protein = food_info['Protein_(g)'] * 2
# 每行数据*-0.5
weighted_fat = -0.5 * food_info['Lipid_Tot_(g)']
# 两数据连接
initial_rating = weighted_protein + weighted_fat
```

#### 基本运算 3

```python
max_calories = food_info['Energ_Kcal'].max()
max_calories = food_info['Energ_Kcal'].min()
```

------

#### 排序

```python
# inplace是否改变本身  true改变本身 默认从小到大排序
food_info.sort_values('Sodium_(mg)', inplace = True)
# 从大到小排序
food_info.sort_values('Sodium_(mg)', inplace = True， ascending = False)
```





#### 
 
#### 

#### 

#### 
















