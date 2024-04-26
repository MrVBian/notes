[TOC]

用户管理软件快捷键的功能，避免代码写得太分散。

# 1.构造方法及单例
私有，无参
```csharp
    private InputManager()
    {
    }
```
单例
```csharp
    private static InputManager mInstance;
    public static InputManager GetInstance()
    {
        if (mInstance == null)
            mInstance = new InputManager();

        return mInstance;
    }
```

# 2.类的成员

## 2.1 Update方法
```csharp
    public override void Update(float deltaTime)
    {
        //机器人和桁架示教走点确认
        if (mEditor.Input.GetKeyUp(KeyCode.Escape)&&!mEditor.Input.IsAnyKey())
        {
            Globals.EventManager.DispatchEvent(EventManager.EventName_KeyCodeUp, KeyCode.Escape);
        }
        //机器人和桁架示教走点取消
        if (mEditor.Input.GetKeyUp(KeyCode.Return)&&!mEditor.Input.IsAnyKey())
        {
            Globals.EventManager.DispatchEvent(EventManager.EventName_KeyCodeUp, KeyCode.Return);
        }
        
        //Scene场景界面全屏或还原
        ClickShowSceneViewFullScreen();
        //快捷键说明界面
        ClickShowKeyboardShortcut();
        //对齐快捷键
        ClickAlignShortcut();
        // ClickVertexSnappingShortcut();
    }
```

机器人和桁架示教走点快捷键功能，场景界面全屏或还原快捷键功能，打开和关闭快捷键说明界面快捷键功能，对齐快捷键功能。