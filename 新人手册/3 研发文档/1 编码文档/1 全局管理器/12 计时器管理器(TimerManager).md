[TOC]


计时器功能
# 1. 构造方法及单例
构造方法，私有，无参
```csharp
    private TimerManager()
    {
    }
```
单例
```csharp
    private static TimerManager mInstance;
    public static TimerManager GetInstance()
    {
        if (mInstance == null)
            mInstance = new TimerManager();
        return mInstance;
    }
```
# 2. MyTimer类
MyTimer类主要用来记录计时器所需数据及计时执行逻辑。

```csharp
//计时器
public class MyTimer
{
    public int loop;//执行 0-n次 -1 无限
    public float interval;//执行间隔
    public bool isDelete;//是否需要删除
    public bool isStop;//是否暂停计时
    public TimerHandler callback;//执行回调

    private float _timer;//计时

    private MyTimer() { }

    public MyTimer(TimerHandler callback, float interval, int loop, bool isStop = false)
    {
        this.callback = callback;
        this.interval = interval;
        this.loop = loop;
        this.isStop = isStop;
        isDelete = false;
        _timer = 0f;
    }
    /// <summary>
    /// 计时逻辑处理
    /// </summary>
    /// <param name="deltaTime"></param>
    /// <returns></returns>
    public bool Tick(float deltaTime)
    {
        if (isStop) return false;
        if (!isDelete)
        {
            if (loop != 0)
            {
                _timer += deltaTime;
                if (_timer >= interval)
                {
                    _timer = 0f;
                    if (loop > 0) loop--;
                    if (loop == 0) isDelete = true;
                    if (callback != null)
                    {
                        callback(this);
                    }

                }
            }
        }
        return isDelete;
    }
}
```
# 3. 类的成员
## 3.1 字段和属性
```csharp
    private bool _isStop;//暂停开关
    private List<MyTimer> _timerList;//计时器列表
```
<center>所有计时器暂停开过和计时器列表</center>

```csharp
    //获取计时器数量
    public int Count
    {
        get { return _timerList.Count; }
    }
```
<center>计时器数量</center>

## 2. 2 成员方法
```csharp
    public override void Init()
    {
        _isStop = true;//计时器初始暂停
        _timerList = new List<MyTimer>();
    }
```
<center>初始化</center>



```csharp
    //注册一个计时器
    public void Regist(TimerHandler callback, float interval, int loop)
    {
        _timerList.Add(new MyTimer(callback, interval, loop));
    }

    public void Regist(TimerHandler callback, float interval)
    {
        Regist(callback, interval, 1);
    }

    public void Regist(MyTimer timer)
    {
        _timerList.Add(timer);
    }
```
注册计时器提供三种方式，其主要是初始计时器(MyTimer)，参数callback完成回调，interval计时时长，loop循环次数(默认为1，-1为无限循环，0为停止)

```csharp
    //删除一个计时器
    public void RemoveTimer(MyTimer timer)
    {
        if (_timerList.Contains(timer)) _timerList.Remove(timer);
    }

    public void RemoveTimer(TimerHandler callback)
    {
        _timerList.ToList().ForEach((mt) =>
        {
            if (mt.callback == callback) _timerList.Remove(mt);
        });
    }
```
<center>删除计时器</center>

```csharp
    //清除所有计时器
    public void RemoveAllTimer()
    {
        _timerList.Clear();
    }
```
<center>清空所有计时器</center>

```csharp
    //播放
    public void Play()
    {
        _isStop = false;
    }
	//暂停
    public void Stop()
    {
        _isStop = true;
    }
```
<center>所有计时器开关</center>

```csharp
    //更新列表计时器
    public override void Update(float deltaTime)
    {
        if (_isStop) return;
        for (int i = _timerList.Count - 1; i >= 0; i--)
        {
            if (_timerList[i].Tick(deltaTime))
            {
                _timerList.Remove(_timerList[i]);
            }
        }
    }
```
<center>更新列表计时器</center>