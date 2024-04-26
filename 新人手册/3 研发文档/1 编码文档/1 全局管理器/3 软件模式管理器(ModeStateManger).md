[TOC]


初始化软件功能模式状态机，向状态机添加不同功能模式状态及功能模式状态切换到其他状态的条件。
# 1. 构造方法及单例
构造方法，私有，无参
```csharp
    private ModeStateManger()
    {
        
    }
```
单例
```csharp
    private static ModeStateManger mInstance;
    public static ModeStateManger GetInstance()
    {
        if (mInstance == null)
            mInstance = new ModeStateManger();

        return mInstance;
    }
```
# 2. 类的成员
## 2. 1 成员方法
```csharp
    public override void Init()
    {
        //初始化模式功能状态机
        fsmSystem = new FSMSystem<ModeStateBase, ModeTransition, ModeStateID>();
        //初始化各种功能模式状态
        mStateList = new List<ModeStateBase>();
        mStateList.Add(new ModeNormalState());
        mStateList.Add(new ModeNestingChartState());
        mStateList.Add(new ModeBehaviorTreeStationState());
        mStateList.Add(new ModeRobotProgrammingState());
        mStateList.Add(new ModeSoftwareProgrammingState());
        mStateList.Add(new ModeSynchronousRobotState());
        mStateList.Add(new ModeBehavioralSignalState());
        mStateList.ForEach((v) =>
        {
            InitModeState(v);
            fsmSystem.AddState(v);
        });

        ModeStateBase _current = mStateList.Find((v) => v.ID == ModeStateID.Normal);
        if (_current != null)
            fsmSystem.SetCurrentState(_current);
    }
```

<center>初始化功能模式状态机及功能模式状态</center>

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

<center>功能模式状态添加切换其他状态条件</center>

```csharp
    /// <summary>
    /// 根据切换条件，切换功能模式状态
    /// </summary>
    /// <param name="transition"></param>
    public void PerformTransition(ModeTransition transition)
    {
        if (fsmSystem.PerformTransition(transition))
        {
            ModeStateChanged?.Invoke();
        }
        else
        {
            Debug.LogWarning("切换模式失败");
        }
    }
```
<center>根据切换条件，切换功能模式状态</center>

```csharp
    /// <summary>
    /// 根据状态ID，判断是否是该功能模式状态
    /// </summary>
    /// <param name="state"></param>
    /// <returns></returns>
    public bool isCurrentState(ModeStateID state)
    {
        return fsmSystem.currentState.ID == state;
    }
```
<center>判断当前是否是状态</center>