[TOC]

# 1. 字段和属性
```csharp
    private IWindowManager mWm;
    //SDK初始化是否成功
    private bool mInitret;
    //机器人和机器人控制器
    private List<RobotModel> mRobotList;
    private List<IKController> mIKList;
    //SDK走点所需参数
    private const int velocity = 100;
    private const int acc = 100;
    private const int smoot = 10;
    //SDK走点数据生成
    List<double> mJoint = new List<double>();
    List<double> mPosition = new List<double>();
    List<double> mRotate = new List<double>();
    //示教走点目标点位
    private Dictionary<string, List<double>> mTarget;
```
# 2. 成员方法
## 2.1 构造方法
```csharp
    public ModeRobotProgrammingState()
    {
        id = ModeStateID.RobotProgramming;

        mWm = IOC.Resolve<IWindowManager>();

        mRobotList = new List<RobotModel>();
        mJoint = new List<double>();
        mPosition = new List<double>();
        mRotate = new List<double>();
        mIKList = new List<IKController>();
        mTarget = new Dictionary<string, List<double>>();
    }
```
## 2.2 DoBeforeEntering
```csharp
    public override void DoBeforeEntering()
    {
        base.DoBeforeEntering();

        Globals.EventManager.DispatchEvent(EventManager.EventName_RobotDragAction,true);
        Globals.EventManager.Regist(EventManager.EventName_SceneModelsChange, SceneModelsChangeEvent);

        SetRobotDebuggingLayout();

        List<RobotModel> _list = Globals.SceneModelsManager.GetModeListByType<RobotModel>(ModelType.Robot);
        if (_list == null || _list.Count == 0) return;

        for (int i = 0; i < _list.Count; i++)
        {
            AddRobot(_list[i]);
        }
    }
```
进入该模式，完成以下逻辑：发出机器人和桁架允许拖拽事件，监听场景内模型变化，设置示教走点界面布局，向SDK添加场景内机器人信息。
## 2.3 Update
```csharp
    /// <summary>
    /// 实时刷新正在示教走点机器人的姿态
    /// </summary>
    /// <param name="deltaTime"></param>
    public override void Update(float deltaTime)
    {
        if (mInitret && mIKList.Count != 0)
        {
            for (int i = 0; i < mIKList.Count; i++)
            {
                IKController _controller = mIKList[i];
                if (mTarget.ContainsKey(_controller.Id))
                {
                    mJoint.Clear();
                    mPosition.Clear();
                    mRotate.Clear();
                    bool _flag = RobotController.isMotion(_controller.Id);
                    if (_flag)
                    {
                        if (RobotController.getStatus(_controller.Id, ref mJoint, 6, ref mPosition, ref mRotate))
                        {
                            _controller.DebuggingMove(mJoint, mTarget[_controller.Id], _flag);
                        }
                    }
                    else
                    {
                        //如有发生瞬移，是isMotion值为false了
                        _controller.DebuggingMove(mJoint, mTarget[_controller.Id], _flag);
                    }
                }
            }
        }
    }
```
实时刷新正在示教走点机器人的姿态
## 2.4 DoBeforeLeaving
```csharp
    public override void DoBeforeLeaving()
    {
        Globals.EventManager.DispatchEvent(EventManager.EventName_RobotDragAction, false);
        Globals.EventManager.UnRegist(EventManager.EventName_SceneModelsChange, SceneModelsChangeEvent);
        if (mInitret)
        {
            RobotController.uninit();
            mInitret = false;
        }
        if (mRobotList != null && mRobotList.Count != 0)
        {
            mRobotList.ForEach((v) => v.ikController.ExitDebugging());
        }
        mRobotList.Clear();
        mTarget.Clear();
        mIKList.Clear();
    }
```
退出该状态之前，先完成清理数据和SDK反初始化

## 2.5 SceneModelsChangeEvent
```csharp
    //监听场景内模型数量的改变
    private void SceneModelsChangeEvent(object[] args)
    {
        if (args.Length != 2||!(args[0] is RobotModel)||!(args[1] is bool)) return;
        RobotModel _model =(RobotModel) args[0];
        bool _flag = (bool)args[1];
        if (_flag)
        {
            AddRobot(_model);
        }
        else
        {
            DeleteRobot(_model);
        }
    }
```
## 2.6 SetRobotDebuggingLayout
```csharp
    //示教走点界面布局
    private void SetRobotDebuggingLayout()
    {
        WindowDescriptor _pointActionWd;
        GameObject _pointActionContent;
        bool _isDialog;

        mWm.CreateWindow("PointAction", out _pointActionWd, out _pointActionContent, out _isDialog);
        LayoutInfo _info = mWm.CreateLayoutInfo(_pointActionContent.transform, _pointActionWd);
        _info.Tab.CanClose = false;
        _info.Tab.IsCloseButtonVisible = false;
        _info.Tab.CanDrag = false;
        _info.Tab.IsOn = false;
        _info.CanMaximize = false;

        mWm.SetLayout(() => LayoutInfo.Horizontal(mWm.CreateLayoutInfo(BuiltInWindowNames.Scene), _info,0.743f));
    }
```
## 2.7 SetMoveData
```csharp
    /// <summary>
    /// 设置需要示教走点的机器人数据
    /// </summary>
    /// <param name="robotID"></param>
    /// <param name="ip"></param>
    /// <param name="jts"></param>
    /// <param name="currentjts"></param>
    public void SetMoveData(string robotID,string ip,List<double> jts,List<double> currentjts)
    {
        if (!mInitret || string.IsNullOrEmpty(robotID) || mRobotList.Count == 0) return;
        for (int i = 0; i < mRobotList.Count; i++)
        {
            RobotModel _model = mRobotList[i];

            if (_model.ikController.Id.Equals(robotID))
            {
                bool _flag= RobotController.setCurrentJoint(robotID,currentjts);
                _flag = RobotController.motion(robotID,jts,velocity,acc,smoot);
                mTarget.Add(robotID, jts);
                mIKList.Add(_model.ikController);
            }
        }
    }
```
## 2.8 AddRobot
```csharp
    /// <summary>
    /// 向SDK添加场景新增机器人数据
    /// </summary>
    /// <param name="model"></param>
    public void AddRobot(RobotModel model)
    {
        
        if (model.ikController == null || string.IsNullOrEmpty(model.ikController.Id)) return;
        if (!mInitret)
        {
            mInitret = RobotController.init();
            Debug.Log("mInitret---->" + mInitret);
        }
        if (!mInitret) return;
        Encoding _encoding = Encoding.GetEncoding("GBK");
        bool _flag=RobotController.addRobot(model.ikController.Id, UtilsLogic.GetStringByEncoding(model.ikController.path, _encoding));
        if (_flag)
        {
            mRobotList.Add(model);
        }
    }
```
## 2.8 DeleteRobot
```csharp
    /// <summary>
    /// 删掉SDK已有机器人数据
    /// </summary>
    /// <param name="model"></param>
    private void DeleteRobot(RobotModel model)
    {
        if (!mInitret||model.ikController==null||string.IsNullOrEmpty(model.ikController.Id)) return;
        RobotController.deleteRobot(model.ikController.Id);
        mRobotList.Remove(model);
        if (mTarget.ContainsKey(model.ikController.Id))
        {
            mTarget.Remove(model.ikController.Id);
        }
    }
```
## 2.9 SetRobotArrived
```csharp
    /// <summary>
    /// 机器人示教走点到达后的数据处理
    /// </summary>
    /// <param name="iKController"></param>
    public void SetRobotArrived(IKController iKController)
    {
        if (mIKList.Contains(iKController))
        {
            mIKList.Remove(iKController);
            if (mTarget.ContainsKey(iKController.Id))
            {
                mTarget.Remove(iKController.Id);
            }
        }
    }
```