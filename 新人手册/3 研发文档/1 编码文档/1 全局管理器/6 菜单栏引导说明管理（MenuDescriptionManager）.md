[TOC]


管理菜单栏引导说明，根据菜单栏的按钮及本地数据和配置表信息，是否弹窗显示该按钮功能说明
# 1. 构造方法及单例
构造方法，私有，无参，读取菜单栏按钮的配置信息数据
```csharp
    /// <summary>
    /// 读取配置信息数据
    /// </summary>
    private MenuDescriptionManager()
    {
        m_localization = IOC.Resolve<ILocalization>();
        if (File.Exists(mConfigPath))
        {
            string _content = File.ReadAllText(mConfigPath);
            mData = LitJson.JsonMapper.ToObject<MenuDescriptionJsonData>(_content);
        }
        else
        {
            mData = new MenuDescriptionJsonData();
            mData.List = new List<MenuDescriptionData>();
        }
    }
```
单例
```csharp
    private static MenuDescriptionManager m_instance;
    public static MenuDescriptionManager GetInstance()
    {
        if (m_instance == null)
            m_instance = new MenuDescriptionManager();
        return m_instance;
    }
```
# 2. 类的成员
## 2.1 成员方法

```csharp
/// <summary>
    /// 比对菜单栏功能和菜单栏引导说明配置信息，再根据用户查看记录，响应是否打开引导界面
    /// </summary>
    public override void Init()
    {
        mEditor = IOC.Resolve<IRTE>() as RuntimeEditor;
        mWm = IOC.Resolve<IWindowManager>();
        mEventDic = new Dictionary<MenuItemInfo, MenuItemEvent>();
        
        //获得软件有哪些菜单栏
        mMenuCreator = mEditor.transform.GetChild(0).Find("MainMenuBar").GetComponent<MenuCreator>();

        Menu[] menus = mMenuCreator.menuPanel.GetComponentsInChildren<Menu>(true);
        for (int i = 0; i < menus.Length; i++)
        {
            if (menus[i].Items.Length == 0) continue;
            for (int j = 0; j < menus[i].Items.Length; j++)
            {
                MenuItemInfo _info = menus[i].Items[j];

                if (_info.Action != null)
                {
                    if (!string.IsNullOrEmpty(_info.Path))
                    {
                        string _key = menus[i].Anchor.name + "/" + _info.Path;
                        //Debug.Log("key---->" + _key);
                        MenuDescriptionData _data = null;
                        for (int k = 0; k < mData.List.Count; k++)
                        {
                            if (mData.List[k].MenuKey.Equals(_key))
                            {
                                _data = mData.List[k];
                                break;
                            }
                        }
                        if (_data == null)
                        {
                            _data = new MenuDescriptionData();
                            _data.MenuKey = _key;
                            string _str= m_localization.GetString(_info.Text);
                            if (string.IsNullOrEmpty(_str))
                            {
                                _data.MenuName = _info.Text;
                            }
                            else
                            {
                                _data.MenuName = _str;
                            }
                            mData.List.Add(_data);
                        }

                        if (_data.imageName == null || _data.imageName.Length == 0)
                            continue;


                        if (!PlayerPrefs.HasKey(_key) || PlayerPrefs.GetInt(_key) == 0)
                        {
                            MenuItemEvent _event = _info.Action;
                            mEventDic.Add(_info, _event);
                            _info.Action = new MenuItemEvent();
                            _info.Action.AddListener((s) => ShowDescriptionView(_data, _info));
                        }
                    }
                }
            }
        }
    }
```

先获得软件的菜单栏功能按钮，然后与配置信息进行比对，如配置信息有相对应的数据，说明该菜单栏按钮有引导说明，再检查本地是否有用户查看引导说明的记录，如果没有，将先保存该菜单功能按钮的事件监听，再重新声明新的事件，添加打开显示菜单栏功能说明界面的事件监听，最后在关闭界面后，响应原有的事件。


```csharp
    /// <summary>
    /// 打开引导说明界面
    /// </summary>
    /// <param name="data"></param>
    /// <param name="info"></param>
    private void ShowDescriptionView(MenuDescriptionData data, MenuItemInfo info)
    {
        mDescriptionView = Globals.ViewManager.ShowDescriptionView();
        mInfo = info;
        DescriptionViewModel _view = mDescriptionView.GetComponent<DescriptionViewModel>();
        _view.Init(data, toggleActin, closeAction);
        mWm.WindowDestroyed += WindowDestroyedEvent;
    }
```

打开引导说明界面的方法，需传入配置信息数据和菜单栏信息数据，根据这两个数据显示引导说明内容。

```csharp
    /// <summary>
    /// 根据用户选择，保存用户查看引导说明的记录
    /// </summary>
    /// <param name="data"></param>
    /// <param name="flag"></param>
    private void toggleActin(MenuDescriptionData data,bool flag)
    {
        MenuItemEvent _event = mEventDic[mInfo];
        if (flag)
        {
            PlayerPrefs.SetInt(data.MenuKey, 0);
            mInfo.Action = new MenuItemEvent();
            mInfo.Action.AddListener((s) => ShowDescriptionView(data, mInfo));
        }
        else
        {
            PlayerPrefs.SetInt(data.MenuKey, 1);
            mInfo.Action = _event;
        }
    }
```

用户操作回调事件，用于保存用户查看引导说明记录。

```csharp
    /// <summary>
    /// 关闭引导说明界面，响应按钮原有事件
    /// </summary>
    /// <param name="obj"></param>
    private void WindowDestroyedEvent(Transform obj)
    {
        if (object.ReferenceEquals(obj, mDescriptionView))
        {
            Sequence seq = DOTween.Sequence();
            seq.AppendInterval(0.1f);
            seq.AppendCallback(() =>
            {
                if (mEventDic.ContainsKey(mInfo))
                {
                    MenuItemEvent _event = mEventDic[mInfo];
                    //mInfo.Action = _event;
                    //mInfo.Action.Invoke(mInfo.Command);
                    _event.Invoke(mInfo.Command);
                }
            });
            seq.SetAutoKill(true);
            seq.SetUpdate(true);

            mWm.WindowDestroyed -= WindowDestroyedEvent;
        }
    }
```
监听软件关闭窗口事件，如关闭引导说明界面，将会响应菜单栏按钮原有的事件。