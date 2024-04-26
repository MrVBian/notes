修改自定义属性的类的界面中某个变量的名称。

# 1. RenameAttribute类
```csharp
    /// <summary>
    /// 改名
    /// </summary>
    public class RenameAttribute : System.Attribute
    {
        public string name
        {
            get;
            private set;
        }
        public RenameAttribute(string name)
        {
            this.name = name;
        }
    }
```

# 2. 使用
```csharp
    [Rename("哈哈")]
    public bool testflag3;
```

![](./imgs/401.png)