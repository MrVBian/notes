[TOC]


用于不同类之间的信息传递。继承ManagerBase
# 1. 构造方法及单例
构造方法，私有，无参
```csharp
    private EventManager()
    {

    }

```
单例
```csharp
    private static EventManager mInstance;

    public static EventManager GetInstance()
    {
        if (mInstance == null)
            mInstance = new EventManager();

        return mInstance;
    }
```
# 2.类的成员
## 2.1 字段和属性
事件名称，公共，常量，string类型：
```csharp
    public const string EventName_RobotDragAction = "RobotDragAction";//机器人拖拽
    public const string EventName_KeyCodeUp = "EventName_KeyCodeUp";//快捷键
    public const string EventName_SelectWindowChange = "EventName_SelectWindowChange";//当前操作窗口修改
    public const string EventName_UpdateWindows = "EventName_UpdateWindows";//更新窗口，传2个参数，第一个为RuntimeWindow，第二个为bool(true为添加，false为删除)
    public const string EventName_SceneModelsChange = "EventName_SceneModelsChange";//更新场景模型，传2个参数，第一个是ModelGameobjectBase，第二个为bool（true为添加，false为删除）
    public const string EventName_AssemblySucceeded = "EventName_AssemblySucceeded";//装配成功，传1个参数，第一个是目标的ModelGameobjectBase,第二个是配件的ModelGameobjectBase
    public const string EventName_SceneViewFullScreen = "EventName_SceneViewFullScreen";
    public const string EventName_RobotGroundTrackMove = "EventName_RobotGroundTrackMove";//地轨移动，传2个参数，IKController,移动数值
    public const string EventName_AssemblyHight = "EventName_AssemblyHight";//装配高亮 第一个参数是类型   第二个参数是开关true是开，false是关
    public const string EventName_AssemblyUpdateChange = "EventName_AssemblyUpdateChange";//所有需要更新的参数都传入
```
## 2.2 成员方法
### 2.2.1 事件注册
```csharp
    /// <summary>
    /// 订阅事件，即存储一组执行事件
    /// </summary>
    /// <param name="eventName">一个触发事件对应的所有执行事件的ID</param>
    /// <param name="handler">执行事件</param>
    public void Regist(string eventName, EventDelegate handler)
    {
        //如果要存储执行事件为空，不处理
        if (handler == null)
        {
            return;
        }

        //如果事件字典中不包含该eventName这个键，则创建一个键值对（即一个新的触发事件）
        if (!mEventListeners.ContainsKey(eventName))
        {
            mEventListeners.Add(eventName, new Dictionary<int, EventDelegate>());
        }

        //获取eventName键对应的值,也同样是一个字典类型
        var handlerDic = mEventListeners[eventName];
        //获取输入的委托事件的哈希值(字典索引的一种方式)
        var handlerHash = handler.GetHashCode();

        //如果输入事件对应的字典中包含执行事件，则从中移除
        if (handlerDic.ContainsKey(handlerHash))
        {
            handlerDic.Remove(handlerHash);
        }

        //将eventName键对应的值（输入的执行事件）的字典类型（用handler的哈希值作为键）加入到事件字典中
        mEventListeners[eventName].Add(handler.GetHashCode(), handler);
    }
```
### 2.2.2 事件触发
```csharp
    /// <summary>
    /// 触发事件：根据ID来找到触发事件对应的所有执行事件并运行
    /// </summary>
    /// <param name="eventName"></param>
    /// <param name="objs"></param>
    public void DispatchEvent(string eventName, params object[] objs)
    {
        // 如果包含eventName这个ID
        if (mEventListeners.ContainsKey(eventName))
        {
            //获取eventName键对应的所有执行事件
            var handlerDic = mEventListeners[eventName];
            if (handlerDic != null && handlerDic.Count > 0)
            {
                var dic = new Dictionary<int, EventDelegate>(handlerDic);
                //通过对eventName键对应的所有执行进行遍历，然后运行
                foreach (var f in dic.Values)
                {
                    try
                    {
                        //执行所有的委托事件，即EventDelegate(object[] args)
                        f(objs);
                    }
                    catch (System.Exception)
                    {

                        throw;
                    }
                }
            }

        }
    }
```
### 2.2.3 事件注销
```csharp
    /// <summary>
    /// 注销事件：在触发事件对应的多个执行事件中删除不再需要的执行事件
    /// </summary>
    /// <param name="eventName"></param>
    /// <param name="handLer"></param>
    public void UnRegist(string eventName, EventDelegate handLer)
    {
        //同样，如果执行事件为空，不进行后续处理
        if (handLer == null)
        {
            return;
        }

        //如果事件字典中包含输eventName这个ID,则移除该键中对应的字典中的输入的执行事件
        if (mEventListeners.ContainsKey(eventName))
        {
            mEventListeners[eventName].Remove(handLer.GetHashCode());
            //在移除后，如果eventName对应的字典为空，或者不存在，则删除该键值对 
            if (mEventListeners[eventName] == null || mEventListeners[eventName].Count == 0)
            {
                mEventListeners.Remove(eventName);
            }


        }
    }
```