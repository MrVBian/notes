[TOC]

# 1. 字段和属性
```csharp
    /// <summary>
    /// 装配位置标签
    /// </summary>
    public const string AssemblePosTag = "AssemblePos";

    private IRTE mEditor;
    private IRuntimeSceneComponent mSceneComponent;
    private RuntimeWindow mScene;

    private GameObject mSelectGameobject;
    /// <summary>
    /// 当前选中模型
    /// </summary>
    private ModelGameobjectBase mFirst;
    //拖拽模型碰撞框属性
    private Collider[] mColliders;
    private bool[] mCollidersEnabled;

    /// <summary>
    /// 可装配模型列表
    /// </summary>
    private List<ModelGameobjectBase> mAssembleModelList;
    /// <summary>
    /// 可装配位置列表
    /// </summary>
    private List<Transform> mPosList;
```
# 2. 成员方法
## 2.1 构造方法
```csharp
    public ToolsAssembleState()
    {
        id = ToolsStateID.Assemble;

        mEditor = IOC.Resolve<IRTE>();
        mPosList = new List<Transform>();
        mAssembleModelList = new List<ModelGameobjectBase>();
    }
```
## 2.2 DoBeforeEntering
```csharp
    public override void DoBeforeEntering()
    {
        mScene = Globals.ViewManager.GetRuntimeWindow(BuiltInWindowNames.Scene)[0];
        mSceneComponent = mScene.IOCContainer.Resolve<IRuntimeSceneComponent>();
        mSceneComponent.SelectionChanging += SelectionChangingEvent;
        mEditor.Selection.SelectionChanged += SelectionChangedEventListener;

        mScene.DropEvent += OnDropEvent;

        Globals.EventManager.Regist(EventManager.EventName_SceneModelsChange, SceneModelsChangeEvent);
        Globals.EventManager.DispatchEvent(EventManager.EventName_AssemblyHight, ModelType.None, true);
        InitFirstSelect();
    }
```
该方法主要是对两种装配方式（点击装配和模型库拖拽装配）所需的事件进行注册监听。

## 2.3 装配方式：模型库拖拽
```csharp
    /// <summary>
    /// 装配：模型库拖拽
    /// </summary>
    /// <param name="args"></param>
    private void SceneModelsChangeEvent(object[] args)
    {
        if (Globals.SceneModelsManager.SceneModelNum == 1 || mEditor.DragDrop.DragObjects == null || mEditor.DragDrop.DragObjects.Length != 1) return;
        if (args.Length != 2) return;
        bool _flag = (bool)args[1];
        ModelGameobjectBase _base = (ModelGameobjectBase)args[0];
        if (_base.transform.parent != null) return;
        mFirst = _base;
        if (_flag)
        {
            mColliders = mFirst.GetComponentsInChildren<Collider>();
            mCollidersEnabled = new bool[mColliders.Length];
            for (int i = 0; i < mColliders.Length; i++)
            {
                mCollidersEnabled[i] = mColliders[i].enabled;
            }

            SetDragModelCollider(true);
            Globals.EventManager.DispatchEvent(EventManager.EventName_AssemblyHight, mFirst.type, true);
            Debug.Log("可装配的改变材质");
        }
        else
        {
            Globals.EventManager.DispatchEvent(EventManager.EventName_AssemblyHight, mFirst.type, false);
            Debug.Log("可装配的恢复材质");
            mFirst = null;
        }
    }
    private void SetDragModelCollider(bool hide)
    {
        if (mColliders == null || mColliders.Length == 0) return;
        for (int i = 0; i < mColliders.Length; i++)
        {
            if (hide)
            {
                mColliders[i].enabled = false;
            }
            else
            {
                mColliders[i].enabled = mCollidersEnabled[i];
            }
        }
    }
    private void OnDropEvent(PointerEventData pointerEventData)
    {
        if (mFirst == null) return;
        if (mScene == null || mScene.Camera == null)
        {
            Debug.LogWarning("场景相机有问题");
            return;
        }
        Ray _ray = mScene.Camera.ScreenPointToRay(Input.mousePosition);

        RaycastHit[] _hits = Physics.RaycastAll(_ray);

        if (_hits != null && _hits.Length != 0)
        {
            //for (int i = _hits.Length - 1; i >= 0; i--)
            for(int i=0;i<_hits.Length;i++)
            {
                Transform _trans = _hits[i].collider.transform;

                if (!_trans.tag.Equals("Terrain"))
                {
                    ModelGameobjectBase _base = GetParentComponent(_trans.gameObject, true);
                    if (_base != null)
                    {
                        if (mFirst.type == _base.type)
                        {
                            Transform _tempparent = _base.transform.parent;
                            //if (_tempparent != null && _tempparent.CompareTag(AssemblePosTag))
                            if (_base.IsAssemble())
                            {
                                ModelGameobjectBase _temp = _tempparent.GetComponentInChildren<ModelGameobjectBase>();
                                if (_temp != null)
                                {
                                    _base = GetParentComponent(_trans.gameObject, false);
                                    SetAssembleItemPos(_tempparent, mFirst, _temp);
                                    mEditor.Selection.activeGameObject = _base.gameObject;
                                    mFirst = _base;
                                    break;
                                }
                            }
                        }
                        _base = null;
                    }
                    _base = GetParentComponent(_trans.gameObject, false);
                    if (_base != null)
                    {
                        mAssembleModelList.Clear();
                        SetAssembleList(mFirst, _base, ref mAssembleModelList);
                        //Debug.Log("count---->" + mAssembleModelList.Count);
                        if (mAssembleModelList.Count > 0)
                        {
                            Transform _parent = null;
                            ModelGameobjectBase _assembleModel = FindAssembleModel(_trans, mFirst, ref _parent);
                            if (_assembleModel != null && _parent != null)
                            {
                                //Debug.Log("可以装配");拖拽装配好
                                SetAssembleItemPos(_parent, mFirst, _assembleModel);
                                mEditor.Selection.activeGameObject = _base.gameObject;
                                mFirst = _base;
                                Globals.EventManager.DispatchEvent(EventManager.EventName_AssemblyHight, _base.type, true);
                                break;
                            }
                            else
                            {
                                //Debug.Log("不可以装配");
                            }
                        }
                        else
                        {
                            //Debug.Log("不可以装配");
                        }
                    }
                }
            }
        }
        SetDragModelCollider(false);
        mCollidersEnabled = null;
        mColliders = null;
    }
```
## 2.4 装配方式：模型点击
```csharp
    /// <summary>
    ///  装配：点击
    /// </summary>
    /// <param name="sender"></param>
    /// <param name="e"></param>
    private void SelectionChangingEvent(object sender, RuntimeSelectionChangingArgs e)
    {
        if (Globals.SceneModelsManager.SceneModelNum == 1)
        {
            mSelectGameobject = null;
            return;
        }

        if (e.Selected.Count != 1)
        {
            mSelectGameobject = null;
        }
        else
        {
            mSelectGameobject = (GameObject)e.Selected[0];
            // Globals.EventManager.DispatchEvent(EventManager.EventName_AssemblyHight, GetParentComponent(mSelectGameobject, true).type, true);
            ModelGameobjectBase _modeBase = mSelectGameobject.GetComponent<ModelGameobjectBase>();
            if (_modeBase != null)
            {
                Globals.EventManager.DispatchEvent(EventManager.EventName_AssemblyHight, _modeBase.type, true);
            }
            else
            {
                if (mSelectGameobject.GetComponentInParent<ModelGameobjectBase>() != null)
                {
                    Globals.EventManager.DispatchEvent(EventManager.EventName_AssemblyHight, mSelectGameobject.GetComponentInParent<ModelGameobjectBase>().type, true);
                }
            }

        }
    }
    private void SelectionChangedEventListener(UnityEngine.Object[] unselectedObjects)
    {
        //Debug.Log("" + Globals.ViewManager.SelectWindow.name);
        if (Globals.ViewManager.SelectWindow.name.Equals(BuiltInWindowNames.Scene) ||
            Globals.ViewManager.SelectWindow.name.Equals(BuiltInWindowNames.Hierarchy))
        {
            if (mEditor.Selection.gameObjects != null && mEditor.Selection.gameObjects.Length == 1)
            {
                GameObject _obj = mEditor.Selection.gameObjects[0];
                if (mFirst == null)
                {
                    mFirst = GetParentComponent(_obj, true);
                }
                else
                {
                    ModelGameobjectBase _temp = GetParentComponent(_obj, true);

                    if (mSelectGameobject != null)
                    {
                        if (object.ReferenceEquals(mFirst, _temp))
                        {
                            return;
                        }
                        else
                        {
                            if (_temp != null && _temp.type == mFirst.type)
                            {
                                //Transform _selectparent = mFirst.transform.parent;
                                bool _flag = false;
                                //if (_selectparent == null || !_selectparent.CompareTag(AssemblePosTag))
                                if(!mFirst.IsAssemble())
                                {
                                    Transform _tempparent = _temp.transform.parent;
                                    //if (_tempparent != null && _tempparent.CompareTag(AssemblePosTag))
                                    if(_temp.IsAssemble())
                                    {
                                        ModelGameobjectBase _parenttemp = _tempparent.GetComponentInParent<ModelGameobjectBase>();
                                        if (_parenttemp != null)
                                        {
                                            _flag = true;
                                            SetAssembleItemPos(_tempparent, mFirst, _parenttemp);
                                            mFirst = GetParentComponent(_parenttemp.gameObject, false);
                                            mEditor.Selection.activeGameObject = mFirst.gameObject;
                                        }
                                    }
                                }
                                if (!_flag)
                                {
                                    mFirst = _temp;
                                }
                            }
                            else
                            {
                                Transform _trans = mSelectGameobject.transform;
                                ModelGameobjectBase _base = GetParentComponent(_obj, false);
                                if (_base == null)
                                {
                                    mFirst = _base;
                                }
                                else
                                {
                                    mAssembleModelList.Clear();
                                    SetAssembleList(mFirst, _base, ref mAssembleModelList);
                                    if (mAssembleModelList.Count > 0)
                                    {
                                        Transform _parent = null;
                                        ModelGameobjectBase _assembleModel = FindAssembleModel(_trans, mFirst, ref _parent);
                                        if (_assembleModel != null && _parent != null)
                                        {
                                            //Debug.Log("可以装配"+_base.name);
                                            SetAssembleItemPos(_parent, mFirst, _assembleModel);
                                            mEditor.Selection.activeGameObject = _base.gameObject;
                                            mFirst = _base;
                                        }
                                        else
                                        {
                                            //Debug.Log("不可以装配");
                                            mFirst = GetParentComponent(_obj, true);
                                        }
                                    }
                                    else
                                    {
                                        //Debug.Log("不可以装配");
                                        mFirst = GetParentComponent(_obj, true);
                                    }
                                }
                            }
                        }
                        mSelectGameobject = null;
                    }
                    else
                    {
                        mFirst = _temp;
                    }
                }
            }
            else
            {
                mFirst = null;
            }

            if (mFirst == null)
            {
                Globals.EventManager.DispatchEvent(EventManager.EventName_AssemblyHight, ModelType.None, true);
            }
            else
            {
                Globals.EventManager.DispatchEvent(EventManager.EventName_AssemblyHight, mFirst.type, true);
            }
        }
    }
```
## 2.5 SetAssembleList
```csharp
    /// <summary>
    /// 设置可装配模型列表数据，包括目标模型及目标模型下其他已装配模型
    /// </summary>
    /// <param name="select"></param>
    /// <param name="target"></param>
    /// <param name="list"></param>
    private void SetAssembleList(ModelGameobjectBase select, ModelGameobjectBase target, ref List<ModelGameobjectBase> list)
    {
        if (target.CanAssemble(select))
        {
            list.Add(target);
        }
        if (target.AssembledItems != null && target.AssembledItems.Count != 0)
        {
            foreach (var transform in target.AssembledItems.Keys)
            {
                if (target.AssembledItems[transform] != null)
                    SetAssembleList(select, target.AssembledItems[transform], ref list);
            }
        }
    }
```
## 2.6 FindAssembleModel
```csharp
    /// <summary>
    /// 查找可装配到的模型及可装配到的位置父节点
    /// </summary>
    /// <param name="transform"></param>
    /// <param name="select"></param>
    /// <param name="parent"></param>
    /// <returns></returns>
    private ModelGameobjectBase FindAssembleModel(Transform transform, ModelGameobjectBase select, ref Transform parent)
    {
        ModelGameobjectBase _returnmodel = null;
        Transform _refparent = null;
        ModelGameobjectBase _returntemp = null;
        Transform _reftemp = null;
        Transform _tempparent = null;
        ModelGameobjectBase _tempmodel = null;
        Transform _temptempparent = null;
        ModelGameobjectBase _temptempmodel = null;
        bool _flag = false;

        mPosList.Clear();
        UtilsLogic.GetTransformByTag(transform, AssemblePosTag, ref mPosList);

        for (int i = 0; i < mAssembleModelList.Count; i++)
        {
            ModelGameobjectBase _model = mAssembleModelList[i];
            if (_model.AssembleItemParents == null || _model.AssembleItemParents.Length == 0) continue;
            for (int j = 0; j < _model.AssembleItemsType.Length; j++)
            {
                if (_model.AssembleItemsType[j] == select.type)
                {
                    Transform _trans = _model.AssembleItemParents[j];
                    for (int k = 0; k < mPosList.Count; k++)
                    {
                        if (object.ReferenceEquals(_trans, mPosList[k]))
                        {
                            if (_model.AssembledItems[mPosList[k]] == null)
                            {
                                _returnmodel = _model;
                                _refparent = mPosList[k];
                            }
                            else
                            {
                                if (_returntemp == null && _reftemp == null)
                                {
                                    _reftemp = mPosList[k];
                                    _returntemp = _model;
                                }
                            }
                            _flag = true;
                            break;
                        }
                    }
                    if (_returnmodel != null && _refparent != null)
                        break;
                    if (_returntemp == null)
                    {
                        if (_model.AssembledItems[_trans] == null)
                        {
                            if (_tempparent == null && _tempmodel == null)
                            {
                                _tempparent = _trans;
                                _tempmodel = _model;
                            }
                        }
                        else
                        {
                            if (_temptempparent == null && _temptempmodel == null)
                            {
                                _temptempparent = _trans;
                                _temptempmodel = _model;
                            }
                        }
                    }
                }
            }
            if (_returnmodel != null && _refparent != null)
                break;
        }
        if (_returnmodel == null)
        {
            if (_returntemp == null)
            {
                if (_tempmodel == null)
                {
                    if (_temptempmodel != null)
                    {
                        _returnmodel = _temptempmodel;
                        parent = _temptempparent;
                    }
                }
                else
                {
                    _returnmodel = _tempmodel;
                    parent = _tempparent;
                }
            }
            else
            {
                _returnmodel = _returntemp;
                parent = _reftemp;
            }
        }
        else
        {
            parent = _refparent;
        }

        if (_returnmodel != null)
        {
            Transform _selectparent = select.transform.parent;
            //if (_selectparent != null && _selectparent.CompareTag(AssemblePosTag))
            if(select.IsAssemble())
            {
                if (_returnmodel.AssembledItems[parent] != null)
                {
                    _returnmodel = null;
                    parent = null;
                }

                if (_flag)
                {
                    for (int i = 0; i < mPosList.Count; i++)
                    {
                        if (object.ReferenceEquals(mPosList[i], _selectparent))
                        {
                            _returnmodel = null;
                            parent = null;
                        }
                    }
                }
                else
                {
                    ModelGameobjectBase _base = GetParentComponent(transform.gameObject, true);
                    if (_base.transform.parent != null && _base.transform.parent.CompareTag(AssemblePosTag))
                    {
                        _returnmodel = null;
                        parent = null;
                    }
                }
            }
        }
        return _returnmodel;
    }
```

## 2.7 SetAssembleItemPos
```csharp
  /// <summary>
    /// 可进行装配的设置
    /// </summary>
    /// <param name="parent"></param>
    /// <param name="select"></param>
    /// <param name="target"></param>
    private void SetAssembleItemPos(Transform parent, ModelGameobjectBase select, ModelGameobjectBase target)
    {
        if (target.AssembleItemParents == null || target.AssembleItemParents.Length == 0) return;

        if (parent != null)
        {
            Transform _selectparent = select.transform.parent;
            //if (_selectparent != null && _selectparent.CompareTag(ToolsAssembleState.AssemblePosTag))
            if(select.IsAssemble())
            {
                ModelGameobjectBase _selectbase = _selectparent.GetComponentInParent<ModelGameobjectBase>();


                if (_selectbase != null && _selectbase.AssembleItemParents != null && _selectbase.AssembleItemParents.Length != 0)
                {
                    if (_selectbase is NestingChartPalletModel)
                    {
                        if (object.ReferenceEquals(_selectbase, target))
                            return;
                        else
                        {
                            NestingChartPalletModel _pallet = _selectbase as NestingChartPalletModel;
                            _pallet.modelList.Remove(select as NestingChartModel);
                        }
                    }
                    else
                    {
                        if (_selectbase.AssembledItems.ContainsKey(_selectparent))
                        {
                            _selectbase.AssembledItems[_selectparent] = null;

                            if (_selectbase is RobotModel)
                            {
                                Globals.EventManager.DispatchEvent(EventManager.EventName_AssemblySucceeded, _selectbase, null);
                            }
                        }
                    }
                }
            }

            Transform _trans = select.transform;
            _trans.SetParent(parent);
            _trans.localEulerAngles = Vector3.zero;
            _trans.localPosition = Vector3.zero;
            _trans.localScale = Vector3.one;

            if (target is NestingChartPalletModel && select is NestingChartModel)
            {
                NestingChartPalletModel _palletModel = target as NestingChartPalletModel;
                _palletModel.modelList.Add(select as NestingChartModel);
            }
            else
            {
                if (target.AssembledItems.ContainsKey(parent) && target.AssembledItems[parent] != null)
                {
                    mEditor.Delete(new GameObject[] { target.AssembledItems[parent].gameObject });
                    Globals.EventManager.DispatchEvent(EventManager.EventName_AssemblyUpdateChange, target );
                    //GameObject.DestroyImmediate(target.AssembledItems[parent].gameObject);
                }
                target.AssembleSuccessful(parent, select);
            }
        }
    }
```
## 2.8 Update
```csharp
    /// <summary>
    /// 模型拆卸操作逻辑
    /// </summary>
    /// <param name="deltaTime"></param>
    public override void Update(float deltaTime)
    {
        if (Input.GetMouseButtonDown(1))
        {
            if (mScene == null || mScene.Camera == null)
            {
                Debug.LogWarning("场景相机有问题");
                return;
            }

            Ray _ray = mScene.Camera.ScreenPointToRay(Input.mousePosition);
            RaycastHit[] _hits = Physics.RaycastAll(_ray);
            if (_hits != null && _hits.Length != 0)
            {
                //for (int i = _hits.Length - 1; i >= 0; i--)
                for(int i=0;i<_hits.Length;i++)
                {
                    Transform _trans = _hits[i].collider.transform;
                    if (_trans.tag.Equals("Terrain")) continue;
                    
                    ModelGameobjectBase _assembleitem = _trans.GetComponentInParent<ModelGameobjectBase>();
                    if (_assembleitem == null) continue;
                    Transform _itemtrans = _assembleitem.transform;
                    Transform _parent = _itemtrans.parent;
                    //if (_parent != null && _parent.CompareTag(AssemblePosTag))
                    if(_assembleitem.IsAssemble())
                    {
                        ModelGameobjectBase _target = _parent.GetComponentInParent<ModelGameobjectBase>();

                        if (_target != null)
                        {
                            if (_target.AssembledItems.ContainsKey(_parent) && object.ReferenceEquals(_target.AssembledItems[_parent], _assembleitem))
                            {
                                object[] _models = _itemtrans.GetComponentsInParent<ModelGameobjectBase>();
                                _target.AssembledItems[_parent] = null;
                                _itemtrans.SetParent(null);
                                ModelGameobjectBase _base = GetParentComponent(_trans.gameObject, false);
                                Transform _basetrans = _base.transform;
                                _itemtrans.position = new Vector3(UnityEngine.Random.Range(_basetrans.position.x + 2, _basetrans.position.x + 3),
                                    0,
                                    UnityEngine.Random.Range(_basetrans.position.z + 2, _basetrans.position.z + 3));

                                _assembleitem.exposeToEditor.CanTransform = _assembleitem.exposeToEditor.CanDrag
                                    = _assembleitem.exposeToEditor.CanDuplicate = _assembleitem.exposeToEditor.CanCreatePrefab = true;
                                mFirst = _assembleitem;
                                mEditor.Selection.activeGameObject = _assembleitem.gameObject;
                                Globals.EventManager.DispatchEvent(EventManager.EventName_AssemblyHight, mFirst.type, true);
                                Globals.EventManager.DispatchEvent(EventManager.EventName_AssemblyUpdateChange, _models);
                                if (_target is RobotModel)
                                {
                                    Globals.EventManager.DispatchEvent(EventManager.EventName_AssemblySucceeded, _target, null);
                                }

                                break;
                            }
                        }
                    }
                }
            }
        }
    }
```
<center>右键点击已装配模型，进行模型拆卸操作。</center>

## 2.9 DoBeforeLeaving
```csharp
    public override void DoBeforeLeaving()
    {
        SetMarkAsDestroyed();

        Globals.EventManager.DispatchEvent(EventManager.EventName_AssemblyHight, ModelType.None, false);
        Debug.Log("关闭装配功能");
        Globals.EventManager.UnRegist(EventManager.EventName_SceneModelsChange, SceneModelsChangeEvent);
        mSceneComponent.SelectionChanging -= SelectionChangingEvent;
        mEditor.Selection.SelectionChanged -= SelectionChangedEventListener;
        mScene.DropEvent -= OnDropEvent;
        mFirst = null;
    }
```
关闭装配功能，注销事件监听。