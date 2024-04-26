[TOC]


处理unity场景之间的切换，主要用于模板场景。
# 1. 构造方法及单例
构造方法，私有，无参
```csharp
    private ScenesManager()
    {
    }
```
单例
```csharp
    private static ScenesManager mInstance;

    public static ScenesManager GetInstance()
    {
        if (mInstance == null)
            mInstance = new ScenesManager();

        return mInstance;
    }
```
# 2. 类的成员
## 2.1 字段
```csharp
    private string mCurrentSceneName;//当前场景名称
    private Action<Scene> mSceneLoadedAction;//场景加载完成回调
```
## 2.2 成员方法
```csharp
    public override void Init()
    {
        mCurrentSceneName = SceneManager.GetActiveScene().name;
        SceneManager.sceneLoaded += sceneLoadedEvent;
    }
```
<center>初始化，获得当前场景名称和添加场景完成事件</center>

```csharp
    /// <summary>
    /// 场景加载完成
    /// </summary>
    /// <param name="arg0"></param>
    /// <param name="arg1"></param>
    private void sceneLoadedEvent(Scene arg0, LoadSceneMode arg1)
    {
        if (mSceneLoadedAction != null)
        {
            mSceneLoadedAction.Invoke(arg0);
            mSceneLoadedAction = null;
        }
    }
    /// <summary>
    /// 加载场景
    /// </summary>
    /// <param name="scanename"></param>
    /// <param name="sceneLoadedAction"></param>
    public void LoadLoadSceneAsync(string scanename, Action<Scene> sceneLoadedAction = null)
    {
        Globals.SceneModelsManager.ClearModel();
        ImportOrExportProjectManager.Instance.projectPath = "";
        //Menu_File.savepath = null;
        mSceneLoadedAction = sceneLoadedAction;
        if (!Globals.ModeStateManger.isCurrentState(ModeStateID.Normal))
            Globals.ModeStateManger.PerformTransition(ModeTransition.NormalEnter);
        SceneManager.LoadSceneAsync(scanename);
    }
```
<center>加载新场景的处理和完成场景加载后的通知回调</center>