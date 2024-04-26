如开发人员，只想对自定义属性面板某个类的界面中，隐藏该类的成员，可使用该标签。

# 1. RuntimeHideInInspector类
```csharp
    /// <summary>
    /// 隐藏
    /// </summary>
    public class RuntimeHideInInspector : System.Attribute
    {
    }
```
# 2. 使用
```csharp
[RuntimeHideInInspector]
public string teststring;
```