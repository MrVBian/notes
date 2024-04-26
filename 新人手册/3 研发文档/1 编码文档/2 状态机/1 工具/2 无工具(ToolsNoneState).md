无工具状态，用于其他工具状态切换，退出使用当前工具。在构造方法定义了状态的ID。

# 1. 类的主体
```csharp
/// <summary>
/// 无工具状态类
/// </summary>
public class ToolsNoneState : ToolsStateBase
{
    public ToolsNoneState()
    {
        id = ToolsStateID.None;
    }
}
```