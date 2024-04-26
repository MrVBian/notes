[TOC]


ModelGameobjectBase是根据类型划分的模型脚本的基类，继承MonoBehaviour。

# 1. 字段和属性
```csharp
    /// <summary>
    /// 可装配类型
    /// </summary>
    protected ModelType[] assembleItemsType;
    public ModelType[] AssembleItemsType
    {
        get => assembleItemsType;
    }
    /// <summary>
    /// 已装配的模型列表
    /// </summary>
    protected Dictionary<Transform, ModelGameobjectBase> assembledItems;
    public Dictionary<Transform, ModelGameobjectBase> AssembledItems
    {
        get => assembledItems;
    }
    /// <summary>
    /// 模型装配点
    /// </summary>
    protected Transform[] assembleItemParents;
    public Transform[] AssembleItemParents
    {
        get => assembleItemParents;
    }
```
# 2. 成员方法
## 2.1 InitAssembleParts
```csharp
    /// <summary>
    /// 初始化装配关系
    /// </summary>
    private void InitAssembleParts()
    {
        if (assembleItemsType != null && assembleItemsType.Length != 0 && assembleItemParents != null && assembleItemParents.Length != 0)
        {
            SetMarkAsDestroyed();

            if (this is NestingChartPalletModel)
            {
                NestingChartPalletModel _pallet = this as NestingChartPalletModel;
                for (int i = 0; i < assembleItemsType.Length; i++)
                {
                    var _models = assembleItemParents[i].GetComponentsInChildren<ModelGameobjectBase>();

                    for (int j = 0; j < _models.Length; j++)
                    {
                        if (_models[j] is NestingChartModel)
                        {
                            _pallet.modelList.Add(_models[j] as NestingChartModel);
                        }
                    }
                }
            }
            else
            {
                for (int i = 0; i < assembleItemsType.Length; i++)
                {
                    if (assembleItemParents[i].childCount == 1)
                    {
                        ModelGameobjectBase _model = assembleItemParents[i].GetComponentInChildren<ModelGameobjectBase>();
                        if (CanAssemble(_model))
                        {
                            AssembleSuccessful(assembleItemParents[i], _model);
                        }
                    }
                    else
                    {
                        if (assembleItemParents[i].childCount > 1)
                        {
                            Debug.LogError("装配点的配件数量有问题");
                        }
                    }
                }
            }
        }
    }
```
## 2.2 AssembleSuccessful
```csharp
    /// <summary>
    /// 装配成功后的装配关系处理
    /// </summary>
    /// <param name="parent"></param>
    /// <param name="model"></param>
    public void AssembleSuccessful(Transform parent, ModelGameobjectBase model)
    {
        if (AssembleItemParents == null || AssembleItemParents.Length == 0 || !AssembledItems.ContainsKey(parent)) return;

        AssembledItems[parent] = model;

        model.exposeToEditor.CanTransform = model.exposeToEditor.CanDrag = model.exposeToEditor.CanDuplicate = model.exposeToEditor.CanCreatePrefab = false;

        if (type == ModelType.Robot && model is RobotGripper)
        {
            RobotGripper _gripper = model as RobotGripper;
            RobotModel _robot = this as RobotModel;
            _gripper.SetChildIKController(_robot.ikController);
        }
        else if (type == ModelType.Truss && model is TrussGripper)
        {
            int _pos = -1;
            for (int i = 0; i < assembleItemParents.Length; i++)
            {
                if (object.ReferenceEquals(parent, assembleItemParents[i]))
                {
                    _pos = i;
                    break;
                }
            }
            TrussGripper _gripper = model as TrussGripper;
            TrussModel _truss = this as TrussModel;
            _gripper.SetChildIKController(_truss.Controller, _pos);
        }
        Globals.EventManager.DispatchEvent(EventManager.EventName_AssemblySucceeded, this, model);
        object[] _models = GetComponentsInParent<ModelGameobjectBase>();
        UpdateChangeMar(_models);
    }
```
## 2.3 CanAssemble
```csharp
    /// <summary>
    /// 判断是否可以装配该模型
    /// </summary>
    /// <param name="model"></param>
    /// <returns></returns>
    public virtual bool CanAssemble(ModelGameobjectBase model)
    {
        if (assembleItemsType == null || assembleItemsType.Length == 0)
            return false;

        for (int i = 0; i < assembleItemsType.Length; i++)
        {
            if (assembleItemsType[i] == model.type)
            {
                return true;
            }
        }
        return false;
    }
```
## 2.4 模型高亮材质设置
```csharp
    public void AddConpoentChangeMaterial()
    {
        _changematerials = new List<ChangeMaterials>();
        meshsTemporary = GetComponentsInChildren<MeshRenderer>();
        for (int i = 0; i < meshsTemporary.Length; i++)
        {
            if (!meshsTemporary[i].gameObject.GetComponent<ChangeMaterials>() && meshsTemporary[i].gameObject.GetComponentInParent<IRuntimeEditor>() == null)
            {
                if (meshsTemporary[i].tag != "Nochange")
                {
                    _changematerials.Add(meshsTemporary[i].gameObject.AddComponent<ChangeMaterials>());
                }
            }
            else if (meshsTemporary[i].gameObject.GetComponent<ChangeMaterials>())
            {
                if (meshsTemporary[i].tag != "Nochange")
                {
                    _changematerials.Add(meshsTemporary[i].gameObject.GetComponent<ChangeMaterials>());
                }
            }
        }
    }
    public void Get_Assemchangematerials()
    {
        meshsTemporary = GetComponentsInChildren<MeshRenderer>();
        ChangeMaterials _change = null;
        for (int i = 0; i < meshsTemporary.Length; i++)
        {
            if (meshsTemporary[i].gameObject.GetComponent<ModelGameobjectBase>() && meshsTemporary[i].gameObject != gameObject)
            {
                return;
            }
            _change = meshsTemporary[i].gameObject.GetComponent<ChangeMaterials>();
            if (_change && meshsTemporary[i].gameObject.GetComponentInParent<IRuntimeEditor>() == null)
            {
                if (meshsTemporary[i].tag != "Nochange")
                {
                    _Assemchangematerials.Add(_change);
                }
            }
        }
    }
```
## 2.5 装配时高亮材质设置逻辑处理
```csharp
/// <summary>
    /// args[0] target  args[1]  select
    /// </summary>
    /// <param name="args"></param>
    private void UpdateChangeMar(object[] args)
    {
        for (int i = 0; i < args.Length; i++)
        {
            _modelselect = args[i] as ModelGameobjectBase;
            if (_modelselect != null)
            {
                _modelselect.AddConpoentChangeMaterial();
            }
        }
    }

    /// <summary>
    /// 判断当前设备是否装配到了某类型模型上
    /// </summary>
    /// <returns>true: 被装配到了某类型模型上; false: 未被装配到某类模型上</returns>
    public bool IsAssemble()
    {
        var parent = transform.parent;
        if (parent == null || parent.CompareTag(ToolsAssembleState.AssemblePosTag) == false)
        {
            return false;
        }
        return true;
    }

    public bool IsHight(ModelType type)
    {
        bool _flag = true;
        if (AssembleItemParents == null || AssembleItemParents.Length == 0) return _flag;

        for (int i = 0; i < assembleItemsType.Length; i++)
        {
            if (assembleItemsType[i] == type)
            {
                if (AssembledItems[AssembleItemParents[i]] == null)
                {
                    _flag = false;
                    return _flag;
                }
            }
        }
        return _flag;
    }

    public bool IsHight()
    {
        bool _flag = true;
        if (AssembleItemParents == null || AssembleItemParents.Length == 0) return _flag;

        for (int i = 0; i < assembleItemsType.Length; i++)
        {
            if (AssembledItems[AssembleItemParents[i]] == null)
            {
                _flag = false;
                return _flag;
            }
        }
        return _flag;
    }
    private void EventName_AssemblyHightEvent(object[] args)
    {
        if (args.Length != 2) return;
        ModelType _type = (ModelType)args[0];
        if (assembleItemsType == null) return;
        bool _flag = (bool)args[1];
        if (_type == ModelType.None)
        {
            if (!IsHight())
            {
                assembleHightOrRecover(_flag);
            }
            return;
        }
        if (!IsHight(_type))
        {
            assembleHightOrRecover(_flag);
        }
        else
        {
            assembleHightOrRecover(false);
        }

    }
    public void assembleHightOrRecover(bool ass)
    {
        if (ass)
        {
            assembleHight();
        }
        else
        {
            assembleRecover();
        }
    }
    private void assembleHight()
    {
        for (int i = 0; i < _Assemchangematerials.Count; i++)
        {
            if (_Assemchangematerials[i])
            {
                _Assemchangematerials[i].AssembleHight();
            }
        }
    }
    public void assembleRecover()
    {
        for (int i = 0; i < _Assemchangematerials.Count; i++)
        {
            if (_Assemchangematerials[i])
            {
                _Assemchangematerials[i].AssembleRecover();
            }
        }
    }
```