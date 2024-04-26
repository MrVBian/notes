[TOC]


# 1. 构造函数
```csharp
    public ToolsVertexSnappingState()
    {
        id = ToolsStateID.VertexSnapping;

        mEditor = IOC.Resolve<IRTE>();
    }
```
定义该状态的ID。
# 2. DoBeforeEntering
```csharp
    public override void DoBeforeEntering()
    {
        base.DoBeforeEntering();
        Debug.Log("开启捕捉功能");

        mEditor.Selection.SelectionChanged += SelectionChangedEvent;

        if (mEditor.Selection.Length == 1)
        {
            if (PositionHandleInput.Instance)
            {
                PositionHandleInput.Instance.IsInVertexSnappingMode = true;
            }
        }
    }
```
进入该状态前调用，如果当前选中模型为1，开启捕捉功能
# 3. SelectionChangedEvent
```csharp
    private void SelectionChangedEvent(UnityEngine.Object[] unselectedObjects)
    {
        if (mEditor.Selection.Length == 1)
        {
            if (PositionHandleInput.Instance)
            {
                PositionHandleInput.Instance.IsInVertexSnappingMode = true;
            }
        }
        else
        {
            if (PositionHandleInput.Instance)
            {
                PositionHandleInput.Instance.IsInVertexSnappingMode = false;
            }
        }
    }
```
选中事件监听，实时开启或关闭捕捉功能。
# 4. DoBeforeLeaving
```csharp
    public override void DoBeforeLeaving()
    {
        base.DoBeforeLeaving();
        mEditor.Selection.SelectionChanged -= SelectionChangedEvent;
        if (PositionHandleInput.Instance)
        {
            PositionHandleInput.Instance.IsInVertexSnappingMode = false;
        }
    }
```
退出该状态，关闭捕捉功能