# matplotlib

--------

#### pandas 日期格式化

```python
import pandas as pd
unrate = pd.read_csv('unrate.csv')
unrate['DATE'] = pd.to_datetime( unrate['DATE'] )
print( unrate.head(12) )
```

--------

#### matplotlib 折线图

```python
import matplotlib.pyplot as plt

# 以'DATE'为横坐标，以'VALUE'为纵坐标
plt.plot( unrate['DATE'], unrate['VALUE'] )

# 横坐标过长时，添加rotation使横坐标文字倾斜显示
plt.xticks(rotation = 45)

# 设置标题
plt.xlabel('横坐标标题')
plt.ylabel('纵坐标标题')
plt.title('大标题')

plt.show()
```

--------

#### 勾画子图 add_subplot

```python
import matplotlib.pyplot as plt

# 画图域 plt.figure
fig = plt.figure()

# 有2*2个子图，第三个参数指定是第几个
ax1 = fig.add_subplot(2, 2, 1)
ax2 = fig.add_subplot(2, 2, 2)
ax3 = fig.add_subplot(2, 2, 4)

plt.show()
```

--------

#### 控制画图域大小，设置子图数据

```python
# 设置画图域大小为3*3，单位英寸
fig = plt.figure( figsize = (3, 3) )
ax1 = fig.add_subplot( 2, 1, 1 )
ax2 = fig.add_subplot( 2, 1, 2 )

# 设置不同子图数据
ax1.plot( np.random.randint(1, 5, 5), np.arange(5) )
ax2.plot( np.arange(10)*3, np.arange(10) )

plt.show()
```

--------

#### 同一图画多条折线

```python
# 日期格式化后，可用dt.month获得月份
unrate['MONTH'] = unrate['DATE'].dt.month
unrate['MONTH'] = unrate['DATE'].dt.month

fig = plt.figure( figsize = (6, 3) )

# 同一图画多条折线，c=‘’设置折线颜色，可用三元组RGB表示
plt.plot( unrate[0:12]['MONTH'], unrate[0:12]['VALUE'], c = 'red' )
plt.plot( unrate[12:24]['MONTH'], unrate[12:24]['VALUE'], c = 'blue' )
```

--------

#### 同一图多条折线缩略图 颜色-折线名

```python
fig = plt.figure( figsize = (10, 6) )
colors = ['red', 'blue', 'green', 'orange', 'black']

for i in range(5):
    start_index = i * 12
    end_index = ( i+1 ) * 12
    # 12项为一组数据
    subset = unrate[start_index : end_index]
    label = str(1948 + i)

    # label 属性即 颜色-[折线名]
    plt.plot( subset['MONTH'], subset['VALUE'], c = colors[i], label = label )

# legend 设置缩略图位置
plt.legend( loc = 'best' )
# print( help(plt.legend) )

# 设置标题
plt.xlabel('横坐标标题')
plt.ylabel('纵坐标标题')
plt.title('大标题')

plt.show()
```

--------

#### matplotlib 柱状图

```python
import pandas as pd
import matplotlib.pyplot as plt
from numpy import arange

num_cols = ['RT', 'ME', 'IM']

bar_heights = data[0, num_cols ].values
bar_positions = arange(3) + 0.75


# fig, ax = plt.subplots 等价于 
# fig = plt.figure() 和 fig.add_subplot(111)
# fig.add_subplot(111) 111对应着(1, 1, 1) 1*1 标号为1子图
fig, ax = plt.subplots()

# 构造多个柱状图
# fig, ax = plt.subplots( nrows = 2, ncols = 2 )
# axes = ax.flatten()
# 把父图分成2*2个子图，ax.flatten()把子图展开赋值给axes,axes[0]便是第一个子图，axes[1]是第二个

# 第一个参数设置横坐标变量名距0距离，第二个参数设置纵坐标高度，第三个参数设置柱状图宽度
ax.bar(bar_positions, bar_heights, 0.3)

tick_positions = arange(1, 6)

# 设置横坐标距0点距离
ax.set_xticks( tick_positions )

# 设置纵坐标高度 第一个参数设置画折线的Series, 第二个参数设置横坐标文字倾斜角度
ax.set_xticklabels( num_cols, rotation = 45 )


# 横向柱状图
# ax.barh(bar_positions, bar_heights, 0.3)
# ax.set_yticks( tick_positions )
# ax.set_yticklabels( num_cols, rotation = 45 )


# 设置横坐标标题
ax.set_xlabel('Rating Source')
# 设置纵坐标标题
ax.set_ylabel('Average Rating')
# 设置大标题
ax.set_title('Average User Rating For Avengers')

plt.show()
```

--------

#### matplotlib 散点图

```python
fig, ax = plt.subplots()

# scatter 第一个参数设置横坐标，第二个参数设置纵坐标，第三个参数设置圆点半径(可省)
ax.scatter( data['横坐标'], data['纵坐标'] )

ax.set_xlabel( '横坐标标题' )
ax.set_ylabel( '纵坐标标题' )

plt.show()
```

--------

#### 构造多个散点图

```python
fig = plt.figure(figsize(5, 10))
ax1 = fig.add_subplot(2, 1, 1)
ax2 = fig.add_subplot(2, 1, 2)

ax1.scatter( data['横坐标'], data['纵坐标'] )
ax1.set_xlabel( '横坐标标题' )
ax1.set_ylabel( '纵坐标标题' )

ax2.scatter( data['横坐标'], data['纵坐标'] )
ax2.set_xlabel( '横坐标标题' )
ax2.set_ylabel( '纵坐标标题' )

plt.show()
```

--------

#### hist 柱形图 

```python
fig, ax = plt.subplots()

# 默认绘图
ax.hist( data['横坐标'] )

# 设置bins分组 bins分组，设置成20，计算机自动合并凑成20个分组
ax.hist( data['横坐标'], bins = 20 )

# 设置只对横坐标值在[4, 5]区间内的建hist柱形图，并且bins = 20
ax.hist( data['横坐标'], range = (4, 5), bins = 20 )
plt.show()
```

--------

#### 多个hist 柱形图

```python
fig = plt.figure( figsize = (5, 20) )

# 构造四个柱形图
ax1 = fig.add_subplot(4, 1, 1)
ax2 = fig.add_subplot(4, 1, 2)
ax3 = fig.add_subplot(4, 1, 3)
ax4 = fig.add_subplot(4, 1, 4)

ax1.hist( data['横坐标'], bins = 20, range = (0, 5) )
ax1.set_title(' 大标题 ')
# 设置纵坐标的范围，横坐标范围在range[0, 5]
ax1.set_ylim( 0, 50 )

ax2.hist( data['横坐标'], bins = 20, range = (0, 5) )
ax2.set_title(' 大标题 ')
# 设置纵坐标的范围，横坐标范围在range[0, 5]
ax2.set_ylim( 0, 50 )

ax3.hist( data['横坐标'], bins = 20, range = (0, 5) )
ax3.set_title(' 大标题 ')
# 设置纵坐标的范围，横坐标范围在range[0, 5]
ax3.set_ylim( 0, 50 )

ax4.hist( data['横坐标'], bins = 20, range = (0, 5) )
ax4.set_title(' 大标题 ')
# 设置纵坐标的范围，横坐标范围在range[0, 5]
ax4.set_ylim( 0, 50 )

plt.show()
```

--------

#### box 盒图

```python
fig, ax = plt.subplots()
ax.boxplot( data['横坐标'] )
# 设置纵坐标的范围
ax.set_ylim( 0, 50 )
plt.show()
```

```python
cols = ['A', 'B', 'C', 'D']
fig, ax = plt.subplots()
ax.boxplot( data[ cols ].values )

# 设置box盒图的横坐标标题以及标题倾斜角
ax.set_xticklabels( cols, rotation = 90 )
# 设置纵坐标的范围
ax.set_ylim(0, 5)

plt.show()
```

--------

#### 细节

```python
fig, ax = plt.subplots()

# 折线-label
ax.plot( data['横坐标'], data['纵坐标'], label = '标题' )

# 设置是否显示数据图参数表提供的对照横线，off为隐藏，默认显示
ax.tick_params( bottom = "off", top = "off", left = "off", right = "off" )

# 设置缩略图位置
ax.legend( loc = "upper right" )

# 设置数据图边界线隐藏
for key, spine in ax.spines.items():
    spine.set_visible( False )

# 颜色三元组 / 255 pandas的三元组
color_dark_blue = ( 0/255, 107/255, 164/255 )

# 调整数据图大小
fig = plt.figure( figsize = ( 12, 12 ) )

# 添加文字
ax.text( 2005, 87, '简述' )
ax.text( 2001, 100, '简述' )

plt.show()
```





