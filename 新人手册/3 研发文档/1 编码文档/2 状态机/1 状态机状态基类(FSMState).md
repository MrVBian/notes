[TOC]


所有状态机状态的虚拟基类，类名为FSMState<T, S>，其中T代表切换状态的条件枚举，S代表某个状态的ID枚举。

# 1. 字段和属性
```csharp
    protected Dictionary<T, S> map = new Dictionary<T, S>();//保存该状态下，切换到其他状态的条件枚举和状态ID枚举
    protected S id;//该状态的ID枚举
    public S ID { get { return id; } }
```
# 2. 虚拟方法
```csharp
    //进入该状态前的逻辑处理
    public abstract void DoBeforeEntering();
    //离开该状态前的逻辑处理
    public abstract void DoBeforeLeaving();
    //该状态中的实时逻辑处理
    public abstract void Update(float deltaTime);
```
# 3. 其他方法
```csharp
    //添加切换到其他状态的条件枚举和状态ID枚举
    public void AddTransition(T trans, S id)
    {
        if (id.Equals(this.ID)) return;
        if (map.ContainsKey(trans))
        {
            Debug.Log("[FSM]:Contains " + trans.ToString());
            return;
        }

        map.Add(trans, id);
    }

    //移除转换条件
    public void RemoveTransition(T trans)
    {
        if (map.ContainsKey(trans))
        {
            map.Remove(trans);
            return;
        }

        Debug.Log("[FSM]:Not have " + trans.ToString());
    }

    //获取输出状态
    public S GetOutputState(T trans)
    {
        if (map.ContainsKey(trans))
        {
            return map[trans];
        }
        return default(S);
    }
```