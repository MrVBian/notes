# seaborn

pip3 install seaborn

--------

#### 整体布局风格设置

```python
import seaborn as sns
import numpy as np
import matplotlib as mpl
import matplotlib.pyplot as plt

# %matplotlib具体作用是当你调用matplotlib.pyplot的绘图函数plot()进行绘图的时候，或者生成一个figure画布的时候，可以直接在你的python console里面生成图像
# 画图的时候，写完代码，直接显示
%matplotlib inline

def sinplot(flip = 1):
    # 等差数列，[0, 14]，找一百个数
    x = np.linspace( 0, 14, 100 )
    for i in range( 1, 7 ):
        plt.plot( x, np.sin(x + i * .5) * ( 7 - i ) * flip )

sinplot()

```

--------

#### 风格

```python
# 使用sinplot默认风格，有五个主题风格
# darkgrid  有方格黑底
# whitegrid 有方格白底
# dark      无方格黑底
# white     无方格白底
# ticks

# 使用默认风格
sns.set()
sinplot()

# 使用whitegrid风格
sns.set_style( "whitegrid" )
data = np.random.normal( size = (20, 6) ) + np.arange(6)

sns.boxplot( data = data )

# 隐藏边框
sns.despine()
```

--------

#### 风格细节设置

```python
# 小提琴绘图
sns.violinplot(data)

# offset 设置中心数据图距离坐标轴距离
sns.despine(offset=100)


sns.set_style( "whitegrid" )
# palette 设置数据内容风格（颜色）为deep
sns.boxplot( data = data, palette = "deep" )

# 隐藏左侧边框
sns.despine( left = True )

# 为不同子图设置不同风格
# with 内一个风格 with外一个风格
with sns.axes_style("darkgrid"):
    plt.subplot(211)
    sinplot()
plt.subplot(212)
sinplot(-1)

# 设置数据图线条风格
# paper
# talk
# poster
# notebook
# font_scale 设置边界数字文字大小，rc = {"lines.linewidth": 4.5}设置线条粗细
sns.set_context( "paper" )
sns.set_context( "notebook", font_scale = 1.5, rc = {"lines.linewidth": 4.5} )
plt.figure( figsize = (8, 6) )
sinplot()
```

--------

#### 调色板 palette

> * color_palette() 能传入任何Matplotlib所支持的颜色
> * color_palette() 不写参数则默认颜色
> * set_palette()   设置所有图的颜色

```python
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
%matplotlib inline
# 设置数据图大小
sns.set( rc = {"figure.figsize": (6, 6)} )

# 获取默认六个颜色
palette = sns.color_palette()
# 画出默认六个颜色
sns.palplot( palette )


# 圆形画板
# 需要六个以上的分类要分区时，在一个圆形颜色空间中画出均匀间隔的颜色，最常用的方法是使用hls的颜色空间，这是RGB值的一个简单转换。
# 获取八个颜色空间
sns.palplot(sns.color_palette( "hls", 8 ))

# 在盒图中使用调色板
data = np.random.normal( size=(20, 8) ) + np.arange(8) / 2
sns.boxplot( data = data, palette = sns.color_palette( "hls", 8 ) )
```

> * hls_palette()函数来控制颜色的亮度和饱和
> * l 亮度 lightness
> * s 饱和 saturation

```python
# 八个hls颜色空间亮度和饱和度设置
sns.palplot( sns.hls_palette( 8, l = .3, s = .8 ) )

# Paired 设置成对具有对比的颜色空间
sns.palplot( sns.color_palette("Paired", 12) )
```

#### 调色板颜色设置

``` python
plt.plot( [0, 1], sns.xkcd_rgb["pale red"], lw = 3 )
```

<++>



