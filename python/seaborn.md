# seaborn

pip3 install seaborn

--------

#### 整体布局风格设置

```python
import seborn as sns
import numpy as np
import matplotlib as mpl
import matplotlib.pyplot as plt

# 画图的时候，写完代码，直接显示
%matplotlib inline

def sinplot(flip = 1):
    x = np.linspace( 0, 14, 100 )
    for i in range( 1, 7 ):
        plt.plot( x, np.sin(x + i * .5) * ( 7 - i ) * flip )

sinplot()

```

--------

#### 

```python
# 使用sinplot默认风格，有五个主题风格
# darkgrid
# whitegrid
# dark
# white
# ticks
sns.set()
sinplot()

sns.set_style( "whitegrid" )
data = np.random.normal( size = (20, 6) ) + np.arange(6)
sns.boxplot( data = data )

sns.despine()

```


