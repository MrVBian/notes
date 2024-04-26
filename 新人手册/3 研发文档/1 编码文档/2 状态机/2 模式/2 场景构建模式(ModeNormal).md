场景构建模式，该模式为软件的默认模式，用于基础构建场景。

# 1. 类的主体
```csharp
/// <summary>
/// 场景构建模式
/// </summary>
public class ModeNormalState : ModeStateBase
{
    private IWindowManager mWm;

    public ModeNormalState()
    {
        id = ModeStateID.Normal;
    }


    public override void DoBeforeEntering()
    {
        if (mWm == null)
        {
            mWm = IOC.Resolve<IWindowManager>();
        }
        else
        {
            base.DoBeforeEntering();
            mWm.SetDefaultLayout();
        }
    }
}
```

定义了改模式的状态ID，进入该模式，将软件界面布局设置为默认布局。