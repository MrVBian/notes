[TOC]


各种工具状态的基类，该基类继承FSMState<ToolsTransition, ToolsStateID>。定义了6个工具状态类，分别为ToolsNoneState，ToolsRobotState，ToolsMeasureState，ToolsAlignState，ToolsVertexSnappingState，ToolsAssembleState。
# 1. 工具状态类ID（ToolsStateID）
```csharp
/// <summary>
/// 工具状态ID
/// </summary>
public enum ToolsStateID
{
    None,//无工具
    Robot,//机器人和桁架拖拽示教
    Measure,//测量
    Align,//对齐
    VertexSnapping,//捕捉
    Assemble//装配
}
```

# 2. 工具转换条件(ToolsTransition)
```csharp
/// <summary>
/// 切换工具状态条件
/// </summary>
public enum ToolsTransition
{
    NoneEnter,//无工具
    RobotEnter,//进入机器人和桁架拖拽示教工具
    MeasureEnter, //进入测距工具
    AlignEnter,//进入对齐工具
    VertexSnappingEnter,//进入捕捉工具
    AssembleEnter,//进入测距工具
}
```
# 3. 工具状态管理
```csharp
public FSMSystem<ToolsStateBase, ToolsTransition, ToolsStateID> fsmSystem;
```
```csharp
//初始化工具状态机
fsmSystem = new FSMSystem<ToolsStateBase, ToolsTransition, ToolsStateID>();
```
# 4. 添加工具状态
```csharp
        mStateList = new List<ToolsStateBase>();
        mStateList.Add(new ToolsNoneState());
        mStateList.Add(new ToolsRobotState());
        mStateList.Add(new ToolsMeasureState());
        mStateList.Add(new ToolsAlignState());
        mStateList.Add(new ToolsVertexSnappingState());
        mStateList.Add(new ToolsAssembleState());
```
# 5. 类的主体
```csharp
/// <summary>
/// 工具状态基类
/// </summary>
public class ToolsStateBase : FSMState<ToolsTransition, ToolsStateID>
{
    public override void DoBeforeEntering()
    {

    }
    public override void Update(float deltaTime)
    {

    }
    public override void DoBeforeLeaving()
    {

    }

    public override void Dispose()
    {

    }
}
```