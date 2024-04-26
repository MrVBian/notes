[TOC]


管理场景中挂载继承模型基类（ModelGameobjectBase）的模型
# 1. 构造方法及单例
构造方法，私有，无参
```csharp
    private SceneModelsManager()
    {
    }
```
单例
```csharp
    private static SceneModelsManager mInstance;

    public static SceneModelsManager GetInstance()
    {
        if (mInstance == null)
            mInstance = new SceneModelsManager();

        return mInstance;
    }
```
# 2. 类的成员
## 2.1 字段和属性
```csharp
    /// <summary>
    /// 场景模型数量
    /// </summary>
    private int mSceneModelNum;

    public int SceneModelNum
    {
        get => mSceneModelNum;
    }
```
<center>记录场景中模型的数量</center>

```csharp
    /// <summary>
    /// 记录场景模型中是否有修改
    /// </summary>
    private bool mIsChanged;
```

<center>记录场景模型中是否有修改，主要用于导出功能</center>

```csharp
    /// <summary>
    /// 记录是否可拖拽
    /// </summary>
    private bool mRobotDragFlag;
```

<center>记录是否可拖拽，主要用于机器人和桁架拖拽</center>

## 2.2 成员方法
### 2.2.1 公共方法(public)
```csharp
    /// <summary>
    /// 注册场景中的模型
    /// </summary>
    /// <param name="model"></param>
    public void RegisterModel(ModelGameobjectBase model)
    {
        string _name = model.gameObject.name.Replace("(Clone)","");
        //根据类型和名称，保存模型
        Dictionary<string, ModelGameobjectBase> _dic = null;
        if (mDic.ContainsKey(model.type))
        {
            _dic = mDic[model.type];
            _name = SetModelName(_name, _dic);
            model.gameObject.name = _name;
            _dic.Add(_name, model);
        }
        else
        {
            _dic = new Dictionary<string, ModelGameobjectBase>();
            _dic.Add(_name,model);
            mDic.Add(model.type, _dic);
        }
        //特殊设置机器人和桁架模型
        if (model is RobotModel)
        {
            RobotModel _robotModel = model as RobotModel;
            SetIKControllerDragFlag(_robotModel.ikController);
        }
        else if (model is TrussModel)
        {
            TrussModel _trussmodel = model as TrussModel;
            if (mRobotDragFlag)
            {
                _trussmodel.Controller.SetDragState(mRobotDragFlag);
            }
        }
        //记录模型个数
        mSceneModelNum += 1;

        Globals.EventManager.DispatchEvent(EventManager.EventName_SceneModelsChange,model,true);
        
        //设置标签，用于导入和导出
        ExposeToEditor _ete = model.exposeToEditor;
        if (_ete != null)
        {
            SetGameObjectSaveTag(_ete);
        }
    }
```
注册场景中的模型，更加模型类型和名称，分类保存模型对象，对场景中的模型进行基本设置，记录模型数量。

```csharp
    /// <summary>
    /// 注销场景中的模型
    /// </summary>
    /// <param name="model"></param>
    /// <param name="isRemove"></param>
    public void UnRegisterModel(ModelGameobjectBase model, bool isRemove = true)
    {
        if (mDic.ContainsKey(model.type))
        {
            Dictionary<string, ModelGameobjectBase> _dic = mDic[model.type];
            string _name = model.gameObject.name;
            if (_dic.ContainsKey(_name))
            {
                GameObjectSaveTag _tag = model.GetComponent<GameObjectSaveTag>();
                if (_tag != null)
                {
                    _tag.flag = false;
                }
                else
                {
                    Debug.LogWarning(model.name+"没有是否导出标记");
                }
                Globals.EventManager.DispatchEvent(EventManager.EventName_SceneModelsChange, model, false);
                if (isRemove)
                {
                    _dic.Remove(_name);
                }
            }

            if (isRemove && _dic.Count == 0)
            {
                mDic.Remove(model.type);
            }
            mSceneModelNum -= 1;
        }
    }
```
将模型从保存列表里中移除,并将其用于导出（GameObjectSaveTag）脚本中flag设置为false。（注意：模型移除的方式是ExposeToEditor中的MarkAsDestroy设置为true）。
```csharp
    /// <summary>
    /// 根据模型类型，获得该类型的列表
    /// </summary>
    /// <typeparam name="T"></typeparam>
    /// <param name="type"></param>
    /// <returns></returns>
    public List<T> GetModeListByType<T>(ModelType type) where T: ModelGameobjectBase
    {
        List<T> _list = null;
        if (mDic.ContainsKey(type))
        {
            _list = new List<T>();

            Dictionary<string, ModelGameobjectBase> _dic = mDic[type];

            foreach (var key in _dic.Keys)
            {
                _list.Add(_dic[key] as T);
            }

        }
        return _list;
    }
```
<center>获得该类型模型的列表</center>

```csharp
    /// <summary>
    /// 获取模型是否有修改
    /// </summary>
    /// <returns></returns>
    public bool GetModelTransformIsChanged()
    {
        return mIsChanged;
    }
    /// <summary>
    /// 设置模型修改为false
    /// </summary>
    public void SetModelTransformChangedIsFalse()
    {
        mIsChanged = false;
    }
```

<center>记录模型空间转换是否有修改</center>

```csharp
    /// <summary>
    /// 注销场景中所有的模型
    /// </summary>
    public void ClearModel()
    {
        foreach (var type in mDic.Keys)
        {
            foreach (var key in mDic[type].Keys)
            {
                UnRegisterModel(mDic[type][key], false);
            }
        }
        mDic.Clear();
        mSceneModelNum = 0;
    }
```

<center>注销场景中所有的模型</center>

### 2.2.2 私有方法(private)事件
```csharp
        mEditor.Object.Started += ObjectStarted;
        ExposeToEditor._MarkAsDestroyedChanged += MarkAsDestroyedChangedEvent;
        ExposeToEditor._TransformChanged += TransformChangedEvent;

        Globals.EventManager.Regist(EventManager.EventName_RobotDragAction, RobotDragEvent);
```
<center>事件注册</center>

```csharp
    /// <summary>
    /// 为非标模型添加模型基类脚本
    /// </summary>
    /// <param name="obj"></param>
    private void ObjectStarted(ExposeToEditor obj)
    {
        if (obj.CanTransform && obj.CanInspect)
        {
            if (obj.GetParent() == null)
            {
                ModelGameobjectBase _base = obj.GetComponent<ModelGameobjectBase>();
                if (_base == null)
                {
                    obj.gameObject.AddComponent<ModelGameobjectBase>();
                }
            }
        }
    }
```
<center>为非标模型添加模型基类脚本</center>

```csharp
    /// <summary>
    /// 监听场景中设置MarkAsDestroyed为true的模型，并处理其装配关系
    /// </summary>
    /// <param name="obj"></param>
    private void MarkAsDestroyedChangedEvent(ExposeToEditor obj)
    {
        ModelGameobjectBase _model = obj.GetComponent<ModelGameobjectBase>();
        if (_model == null) return;
        List<ModelGameobjectBase> _list = new List<ModelGameobjectBase>();
        GetAssembleModels(_model, ref _list);
        if (obj.MarkAsDestroyed)
        {
            UnRegisterModel(_model);
            if (_list.Count != 0)
                _list.ForEach((v) => UnRegisterModel(v));
        }
        else
        {
            RegisterModel(_model);

            if (_list.Count != 0)
                _list.ForEach((v) => RegisterModel(v));
        }
        //处理装配关系
        Transform _selectparent = _model.transform.parent;
        //if (_selectparent != null && _selectparent.CompareTag(ToolsAssembleState.AssemblePosTag))
        if(_model.IsAssemble())
        {
            ModelGameobjectBase _selectbase = _selectparent.GetComponentInParent<ModelGameobjectBase>();
            if (_selectbase != null && _selectbase.AssembleItemParents != null && _selectbase.AssembleItemParents.Length != 0)
            {
                if (_selectbase is NestingChartPalletModel && _model is NestingChartModel)
                {
                    NestingChartPalletModel _pallet = _selectbase as NestingChartPalletModel;
                    if (obj.MarkAsDestroyed)
                    {
                        _pallet.modelList.Remove(_model as NestingChartModel);
                    }
                    else
                    {
                        _pallet.modelList.Add(_model as NestingChartModel);
                    }
                }
                else
                {
                    if (_selectbase.AssembledItems.ContainsKey(_selectparent))
                    {
                        if (obj.MarkAsDestroyed)
                        {
                            _selectbase.AssembledItems[_selectparent] = null;

                            if (_selectbase is RobotModel)
                            {
                                Globals.EventManager.DispatchEvent(EventManager.EventName_AssemblySucceeded, _selectbase, null);
                            }
                        }
                        else
                        {
                            ModelGameobjectBase _assembledmodel = _selectbase.AssembledItems[_selectparent];
                            if (_assembledmodel != null)
                                GameObject.DestroyImmediate(_assembledmodel.gameObject);

                            _selectbase.AssembleSuccessful(_selectparent, _model);
                        }
                    }
                }
            }
        }
    }
```
监听场景模型挂载脚本ExposeToEditor的MarkAsDestroyed，并处理其装配关系(撤销，反撤销，删除，销毁模型)

```csharp
 /// <summary>
    /// 设置机器人和桁架的为可拖拽
    /// </summary>
    /// <param name="args"></param>
    private void RobotDragEvent(object[] args)
    {
        if (args.Length != 1 || !(args[0] is bool)) return;

        mRobotDragFlag = (bool)args[0];
        mSelection.activeObject = null;

        List<RobotModel> _list = GetModeListByType<RobotModel>(ModelType.Robot);
        
        if (_list != null && _list.Count != 0)
        {
            _list.ForEach((v) => {
                if (v.ikController == null)
                    Debug.Log("1111111111");
                SetIKControllerDragFlag(v.ikController);
            });
        }

        List<TrussModel> _trusslist= GetModeListByType<TrussModel>(ModelType.Truss);
        if (_trusslist != null && _trusslist.Count != 0)
        {
            _trusslist.ForEach((v) =>
            {
                v.Controller.SetDragState(mRobotDragFlag);
            });
        }
    }
    private void SetIKControllerDragFlag(IKController ik)
    {
        ik.SetDragState(mRobotDragFlag);
        ik.SetCamera(mRobotDragFlag);
        if (mRobotDragFlag)
        {
            ik.RobotDragAction += RobotDragShowEvent;
        }
        else
        {
            RobotDragShowEvent(null, null);
            ik.RobotDragAction -= RobotDragShowEvent;
        }
    }

    private void RobotDragShowEvent(Transform target, Transform self)
    {
        //mSceneViewModel?.RobotDragShow(target, self);
    }
```
<center>处理机器人和桁架末端拖拽事件</center>