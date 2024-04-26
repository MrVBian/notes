[TOC]


用于点击拖拽机器人或者桁架末端，改变设备姿态的工具。该类继承ToolsStateBase。
# 1. 字段和属性
```csharp
    private IWindowManager mWm;
    private List<SBSceneViewModel> mSceneViewModel;//Scene界面model列表,有可能会存在多个Scene界面

    private IKController mCurrentController;//当前选中机器人控制器
    private Transform mVector3EditorTarget;//用于RuntimeEditor的Vector3Editor UI预设
```
# 2. 成员方法
## 2.1 构造方法
```csharp
    public ToolsRobotState()
    {
        id = ToolsStateID.Robot;//该工具状态ID
        mWm = IOC.Resolve<IWindowManager>();
    }
```
## 2.2 DoBeforeEntering（）
```csharp
    public override void DoBeforeEntering()
    {
        //SpeedBuilderController.RobotDragAction?.Invoke(true);
        Globals.EventManager.DispatchEvent(EventManager.EventName_RobotDragAction, true);

        mSceneViewModel = Globals.ViewManager.GetSceneViewModels();

        if (mSceneViewModel != null && mSceneViewModel.Count != 0)
        {
            mVector3EditorTarget = new GameObject().transform;
            GameObject.DontDestroyOnLoad(mVector3EditorTarget);
            mVector3EditorTarget.gameObject.AddComponent<RTSLIgnore>();

            mSceneViewModel.ForEach((v) => InitSceneWindow(v));
        }

        Globals.EventManager.Regist(EventManager.EventName_UpdateWindows, UpdateWindowsEvent);
    }
```
进入该状态之前调用，发出运行机器人或桁架可拖拽事件，获得当前Scene场景界面列表，初始化Scene场景界面，注册监听Scene场景界面修改。
## 2.3 UpdateWindowsEvent（）
```csharp
    private void UpdateWindowsEvent(object[] args)
    {
        if (args.Length != 2&&!(args[0] is RuntimeWindow)) return;

        RuntimeWindow _window = (RuntimeWindow)args[0];
        if (!_window.name.Equals(BuiltInWindowNames.Scene)) return;
        bool _flag = (bool)args[1];
        SBSceneViewModel _view = _window.GetComponent<SBSceneViewModel>();
        if (_flag)
        {
            mSceneViewModel.Add(_view);
            InitSceneWindow(_view);
        }
        else
        {
            if (mSceneViewModel.Contains(_view))
            {
                mSceneViewModel.Remove(_view);
            }
        }
    }
```
监听界面修改，如Scene界面有改变，则修改Scene界面Model列表。
## 2.4 InitSceneWindow
```csharp
    private void InitSceneWindow(SBSceneViewModel viewmodel)
    {

        viewmodel.robotWindow.gameObject.SetActive(true);
        viewmodel.robotWindow.OpenExportBtn.onClick.AddListener(openExportWindow);
        viewmodel.robotWindow.OpenImportBtn.onClick.AddListener(openImportWindow);
        viewmodel.robotWindow.ConfirmBtn.gameObject.SetActive(false);
        viewmodel.robotWindow.ConfirmBtn.onClick.AddListener(()=>robotEditorConfirmListener(viewmodel));


        viewmodel.robotWindow.PosEditor.Init("position", mVector3EditorTarget, Strong.MemberInfo((Transform x) => x.position),true,null,null,null,null,null,null,
            ()=> robotEditorConfirmListener(viewmodel));
        viewmodel.robotWindow.RotEditor.Init("rotation", mVector3EditorTarget, Strong.MemberInfo((Transform x) => x.localScale), true, null, null, null, null, null, null, 
            () => robotEditorConfirmListener(viewmodel));

        if (mCurrentController == null)
        {
            viewmodel.robotWindow.mainWindow.SetActive(false);
        }
        else
        {
            viewmodel.robotWindow.mainWindow.SetActive(true);
            SetVectorEditorShow();
        }
    }
```
设置该状态下，场景界面的显示，设置按钮事件响应和末端坐标显示。

## 2.5 openExportWindow和openImportWindow
```csharp
    /// <summary>
    /// 打开导出界面
    /// </summary>
    private void openExportWindow()
    {
        Transform _trans = mWm.CreateDialogWindow("ExportAndImport", "ExportWindw", null, null, false, 600, 340);
        ExportAndImportViewModel _model = _trans.GetComponent<ExportAndImportViewModel>();
        // _model.Init(2, mRobotIK, mBioIKController);
        _model.Init(2, mCurrentController);
    }

    /// <summary>
    /// 打开导入界面
    /// </summary>
    private void openImportWindow()
    {
        Transform _trans = mWm.CreateDialogWindow("ExportAndImport", "ImportWindw", null, null, false, 600, 340);
        ExportAndImportViewModel _model = _trans.GetComponent<ExportAndImportViewModel>();
        //_model.Init(1,mRobotIK,mBioIKController);
        _model.Init(1, mCurrentController);
    }
```
<center>场景界面按钮的事件响应</center>
## 2.6 SetIKController
```csharp
    /// <summary>
    /// 设置当前选中机器人控制器
    /// </summary>
    /// <param name="controller"></param>
    public void SetIKController(IKController controller)
    {
        if (mCurrentController != null)
        {
            mCurrentController.RobotDragFlagAction -= RobotDragAction;
        }
        mCurrentController = controller;
        if (mSceneViewModel == null||mSceneViewModel.Count==0) return;
        if (controller == null)
        {
            //mRobotWin.SetActive(false);
            mSceneViewModel.ForEach((v) => v.robotWindow.mainWindow.SetActive(false));
        }
        else
        {
            mSceneViewModel.ForEach((v) => v.robotWindow.mainWindow.SetActive(true));  
            mCurrentController.RobotDragFlagAction += RobotDragAction;
            SetVectorEditorShow();
        }
    }
```
## 2.7 机器人用户自定义窗口设置
```csharp
    /// <summary>
    /// 拖拽机器人，刷新机器人用户自定义窗口事件
    /// </summary>
    private void RobotDragAction()
    {
        SetVectorEditorShow();
    }
    /// <summary>
    /// 刷新机器人用户自定义窗口事件
    /// </summary>
    private void SetVectorEditorShow()
    {
        if (mCurrentController == null||mSceneViewModel==null||mSceneViewModel.Count==0) return;
        Vector3 _pos = mCurrentController.GetTargetPositionForShow();
        Vector3 _ang = mCurrentController.GetTargetEulerAnglesForShow();

        mSceneViewModel.ForEach((v)=>    
        {
            v.robotWindow.PosEditor.SetValue(_pos);
            v.robotWindow.PosEditor.Reload(true);
            v.robotWindow.RotEditor.SetValue(_ang);
            v.robotWindow.RotEditor.Reload(true);
        });
    }
    /// <summary>
    /// 根据用户的输入，设置机器人的姿态
    /// </summary>
    private void robotEditorConfirmListener(SBSceneViewModel viewmodel)
    {
        if (mCurrentController == null) return;
        Vector3 _pos = viewmodel.robotWindow.PosEditor.GetValue();
        Vector3 _ang = viewmodel.robotWindow.RotEditor.GetValue();
        mCurrentController.SetRobotByShowData(_pos, _ang);
    }
```
## 2.8 DoBeforeLeaving
```csharp
    public override void DoBeforeLeaving()
    {
        //SpeedBuilderController.RobotDragAction?.Invoke(false);
        Globals.EventManager.DispatchEvent(EventManager.EventName_RobotDragAction, false);
        Globals.EventManager.UnRegist(EventManager.EventName_UpdateWindows, UpdateWindowsEvent);

        if (mSceneViewModel != null && mSceneViewModel.Count != 0)
        {
            mSceneViewModel.ForEach((v) =>
            {
                v.robotWindow.gameObject.SetActive(false);
                v.robotWindow.OpenExportBtn.onClick.RemoveAllListeners();
                v.robotWindow.OpenImportBtn.onClick.RemoveAllListeners();
                v.robotWindow.ConfirmBtn.onClick.RemoveAllListeners();
            });
            mSceneViewModel.Clear();
        }
        GameObject.Destroy(mVector3EditorTarget.gameObject);
    }
```
退出该状态前的逻辑处理，主要是移除事件，设置场景界面