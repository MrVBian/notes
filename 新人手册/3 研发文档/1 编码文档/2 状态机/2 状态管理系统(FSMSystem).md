[TOC]


该类用于管理不同类型状态机，类名为：FSMSystem<F, T, S>，F为继承FSMState<T, S>的状态基类，T为切换状态的条件枚举，S代表某个状态的ID枚举。

# 1. 字段和属性
```csharp
    private List<F> _states;//状态列表
    private F _currentState;//当前状态
    public F currentState { get { return _currentState; } }
    private F _previousState;//前一状态
    public F previousState { get { return _previousState; } }
    private F _globalState;//全局状态
    public F globalState { get { return _globalState; } }
```
# 2. 主要方法
```csharp
    //添加一个状态
    public void AddState(F state)
    {
        for (int i = _states.Count - 1; i >= 0; i--)
        {
            if (_states[i].ID.Equals(state.ID))
            {
                Debug.Log("[FSM]:Contains " + state.ID.ToString());
                return;
            }
        }
        _states.Add(state);
    }
```

<center>向状态列表添加继承F的状态类对象</center>

```csharp
    public bool PerformTransition(T trans)
    {
        S id = _currentState.GetOutputState(trans);

        for (int i = _states.Count - 1; i >= 0; i--)
        {
            if (_states[i].ID.Equals(id))
            {
                changeState(_states[i]);
                return true;
            }
        }
        return false;
    }
```

<center>根据当前状态下添加的切换条件及状态ID，改变状态</center>

```csharp
    private void changeState(F t)
    {
        _previousState = _currentState;//改变前一状态

        _currentState = t;//改变当前状态

        _previousState.DoBeforeLeaving();//离开

        _currentState.DoBeforeEntering();//进入
    }
```
<center>处理改变状态时的逻辑</center>

```csharp
    //回到前一状态
    public void BackToPreviousState()
    {
        if (_previousState != null) changeState(_previousState);
    }

    //初始化状态
    public void SetCurrentState(F t)
    {
        _currentState = t;
        _currentState.DoBeforeEntering();
    }

    public void SetPreviousState(F t)
    {
        _previousState = t;
    }

    public void SetGlobalState(F t)
    {
        _globalState = t;
    }

    //发送消息 群发 单发
    public void DispatchMessage(FSMMessage msg)
    {
        for (int i = _states.Count - 1; i >= 0; i--)
        {
            _states[i].MessageHandler(msg);
        }
    }

    public void DispatchMessage(S id, FSMMessage msg)
    {
        for (int i = _states.Count - 1; i >= 0; i--)
        {
            if (_states[i].ID.Equals(id))
            {
                _states[i].MessageHandler(msg);
                break;
            }
        }
    }
```
<center>其他方法</center>