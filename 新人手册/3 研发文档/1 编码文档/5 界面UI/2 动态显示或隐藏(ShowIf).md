[TOC]


根据条件，设置类的成员在自定义属性面板该类界面中的显示或隐藏。
# 一. ShowIf类
```csharp
    public class ShowIf : System.Attribute
    {
        public string Name
        {
            get;
            private set;
        }
        public object[] Value
        {
            get;
            private set;
        }

        public ShowIf(string name, params object[] value)
        {
            Name = name;
            Value = value;
        }
    }
```
# 二. 使用方法
## 1. 值类型
```csharp
    [ShowIf("testflag1", true)]
    public int show1;

    [ShowIf("testint", 1)]
    public int show3;

    [ShowIf("testfloat", 0f)]
    public int show4;

    [ShowIf("testenum", TestEnum.Test1)]
    public int show5;
```
通过比对该类中某个成员的值，动态显示和隐藏该成员，需传两个值，一个是某个成员的名称，一个是显示该成员的条件（值相等）。

## 2. “Or”和“And”
```csharp
    [ShowIf("Or", "testflag2", "testflag3")]
    public int show2;

    [ShowIf("And", "testflag1", "testflag3")]
    public int show6;
```
当第一个字符串为“Or”或“And”，且传入的其他参数多个bool类型成员的名称时，显示和隐藏的条件为对多个bool类型成员执行“Or”或“And”逻辑处理.