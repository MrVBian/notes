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

# 查找第80个样例参数age的值
data.loc[80, "age"]
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
# 要排序的是哪项，第一个参数
food_info.sort_values('Sodium_(mg)', inplace = True)

# 从大到小排序
food_info.sort_values('Sodium_(mg)', inplace = True， ascending = False)

# 排序并更改索引
new_data = data.sort_values('age', ascending = False)
# print(new_data.loc[0:10]) # 按从大到小排列，下标为原数据下标

# 下标重新排列，不再是原数据下标，drop = True ：直接修改本元素
new_data = new_data.reset_index(drop = True)
# print(new_data.loc[0:10]) # 按从大到小排列，下标为1-10
```

--------

#### 缺失参数的值

```python
age = data["age"]
# print(age.loc[0:10])

# 找到age中缺失参数的下标
age_is_null = pd.isnull(age)
# print(age_is_null)

# 找到age中缺失参数的值
age_null_true = age[age_is_null]
# print(age_null_true)

# 缺失参数的值的数量
age_null_count = len(age_null_true)
# print(age_null_count)

# 将缺失参数的样例丢弃
# dropna() 将subset里面缺失参数的样例丢弃
new_sample = data.dropna(axis = 0, subset = ["age", Sex])

```
--------

#### 平均值计算

```python
age = data["age"][age_is_null == False]
age_mean = sum(age) / len(age)

age_mean = data["age"].mean()

# pivot_table函数
# 以Pclass为键，以Survived为操作数，求np.mean均值,缺省aggfunc时默认是求均值
passage_survival = data.pivot_table(index = "Pclass", values = "Survived", aggfunc = np.mean)

# 以index为键，以values为操作数，求np.sum和
port_stats = data(index = "Embarked", values = ["Fare", "Survived"], aggfunc = np.sum)

```
--------

#### 自定义函数
 
```python
# apply 函数，自定义函数绑定
# sample 1
def not_null_count(column):
    column_null = pd.isnull(column)
    null = column[column_null]
    return len(null)

column_null_count = data.apply(not_null_count)
print(column_null_count)


# sample 2
def which_class(row):
    pclass = row['Pclass']
    if pd.isnull(Pclass):
        return "Unknown"
    elif pclass == 1:
        return "First Class"
    elif pclass == 2:
        return "Second Class"
    elif pclass == 3:
        return "Third Class"

classes = data.apply(which_class, axis = 1)
print(classes)
# 结果
# 0 First Class
# 3 Third Class


# sample 3
def generate_age_label(row):
    age = row['age']
    if pd.isnull(age):
        return "unknown"
    elif age < 18:
        return "minor"
    else:
        return "adult"

age_labels = data.apply(generate_age_label, axis = 1)
print(age_labels)
# 结果
# 0 adult
# 1 unknown
```

<++>

#### 

#### 

#### 
















