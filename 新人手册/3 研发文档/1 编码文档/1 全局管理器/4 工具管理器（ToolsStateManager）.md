[TOC]


初始化软件工具状态机，向状态机添加不同工具状态及工具状态切换到其他状态的条件。
# 1. 构造方法及单例
构造方法，私有，无参
```csharp
    private ToolsStateManager()
    {  
    }
```
单例
```csharp
    private static ToolsStateManager mInstance;
    public static ToolsStateManager GetInstance()
    {
        if (mInstance == null)
            mInstance = new ToolsStateManager();

        return mInstance;
    }
```
# 2. 类的成员
## 2.1 成员方法
```csharp
    public override void Init()
    {
        fsmSystem = new FSMSystem<ToolsStateBase, ToolsTransition, ToolsStateID>();

        mStateList = new List<ToolsStateBase>();
        mStateList.Add(new ToolsNoneState());
        mStateList.Add(new ToolsRobotState());
        mStateList.Add(new ToolsMeasureState());
        mStateList.Add(new ToolsAlignState());
        mStateList.Add(new ToolsVertexSnappingState());
        mStateList.Add(new ToolsAssembleState());

        mStateList.ForEach((v) =>
        {
            InitFunctionState(v);
            fsmSystem.AddState(v);
        });

        // 初始工具状态：装配功能开启(场景构建模式下默认开启装配功能)
        ToolsStateBase _current = mStateList.Find((v) => v.ID == ToolsStateID.None);
        if (_current != null)
            fsmSystem.SetCurrentState(_current);
    }
```
<center>初始化工具状态机及工具状态</center>

```csharp
    /// <summary>
    ///  每个工具状态添加切换条件
    /// </summary>
    /// <param name="state"></param>
    private void InitFunctionState(ToolsStateBase state)
    {
        state.AddTransition(ToolsTransition.NoneEnter, ToolsStateID.None);
        state.AddTransition(ToolsTransition.RobotEnter, ToolsStateID.Robot);
        state.AddTransition(ToolsTransition.MeasureEnter, ToolsStateID.Measure);
        state.AddTransition(ToolsTransition.AlignEnter, ToolsStateID.Align);
        state.AddTransition(ToolsTransition.VertexSnappingEnter, ToolsStateID.VertexSnapping);
        state.AddTransition(ToolsTransition.AssembleEnter, ToolsStateID.Assemble);
    }
```

<center>工具状态添加切换其他状态条件</center>

```csharp
    /// <summary>
    /// 根据切换条件，切换工具状态
    /// </summary>
    /// <param name="transition"></param>
    public void PerformTransition(ToolsTransition transition)
    {
        if (fsmSystem.PerformTransition(transition))
        {
            ToolsStateChanged?.Invoke();
        }
        else
        {
            Debug.LogWarning("切换工具失败");
        }
    }
```
<center> 根据切换条件，切换工具状态</center>

```csharp
    /// <summary>
    /// 根据状态ID，判断是否是该工具状态
    /// </summary>
    /// <param name="state"></param>
    /// <returns></returns>
    public bool isCurrentState(ToolsStateID state)
    {
        return fsmSystem.currentState.ID == state;
    }
```
<center>根据状态ID，判断当前是否是该状态</center>