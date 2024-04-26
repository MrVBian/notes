监听某个参数在自定义属性的类的界面中值的修改。

# 1. OnValueChangedAttribute类
```csharp
    /// <summary>
    /// 监听面板值的修改
    /// </summary>
    public class OnValueChangedAttribute: System.Attribute
    {
        public string eventName;

        public OnValueChangedAttribute(string name)
        {
            eventName = name;
        }
    }
```
需传一个参数，为函数名称。
# 1. 使用
```csharp
    [BoxGroup("哈哈哈", typeof(bool)),OnValueChanged("hahaha"),TipsPanel("hahaha", TipsType.Normal)]
    public bool testflag1;
```

```csharp
    public void hahaha(PropertyDescriptor descriptor)
    {
        if (descriptor.MemberInfo.Name.Equals(nameof(testflag1)))
        {
            Debug.Log("OnValueChanged---->"+ descriptor.MemberInfo.Name+":"+ testflag1);
        }
    }
```