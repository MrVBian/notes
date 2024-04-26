[TOC]


各种功能模式状态的基类，该基类继承FSMState<ModeTransition, ModeStateID>。定义了4个功能模式状态类，分别为：ModeNormalState，ModeRobotProgrammingState，ModeSynchronousRobotState，ModeBehavioralSignalState。
# 1. 功能模式状态ID(ModeStateID)
```csharp
/// <summary>
/// 功能模式状态ID
/// </summary>
public enum ModeStateID
{
    Normal,//场景构建
    NestingChart,
    RobotProgramming,//示教走点
    SoftwareProgramming,
    BehaviorTreeStation,
    SynchronousRobot,//数字孪生
    BehavioralSignal//行为信号
}
```
# 2. 功能模式切换条件(ModeTransition)
```csharp
/// <summary>
/// 切换功能模式状态条件
/// </summary>
public enum ModeTransition
{
    NormalEnter,//进入场景构建
    NestingChartEnter,
    RobotProgrammingEnter,//进入示教走点
    SoftwareProgrammingEnter,
    BehaviorTreeStationEnter,
    SynchronousRobotEnter,//进入数字孪生
    BehavioralSignalEnter//进入行为信号
}
```
# 3. 功能状态管理
```csharp
public FSMSystem<ModeStateBase, ModeTransition, ModeStateID> fsmSystem;
```
```csharp
fsmSystem = new FSMSystem<ModeStateBase, ModeTransition, ModeStateID>();
```
# 4. 添加功能模式状态
```csharp
        mStateList = new List<ModeStateBase>();
        mStateList.Add(new ModeNormalState());
        mStateList.Add(new ModeNestingChartState());
        mStateList.Add(new ModeBehaviorTreeStationState());
        mStateList.Add(new ModeRobotProgrammingState());
        mStateList.Add(new ModeSoftwareProgrammingState());
        mStateList.Add(new ModeSynchronousRobotState());
        mStateList.Add(new ModeBehavioralSignalState());
```
# 5. 类的主体
```csharp
/// <summary>
/// 功能模式状态基类
/// </summary>
public class ModeStateBase : FSMState<ModeTransition, ModeStateID>
{
    public override void DoBeforeEntering()
    {
        // 模式切换时：默认将工具切回至None
        if (!Globals.ToolsStateManager.isCurrentState(ToolsStateID.None))
        {
            Globals.ToolsStateManager.PerformTransition(ToolsTransition.NoneEnter);
        }
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