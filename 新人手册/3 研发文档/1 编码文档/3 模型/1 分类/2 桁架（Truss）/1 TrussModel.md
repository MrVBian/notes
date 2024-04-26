桁架模型类，继承ModelGameobjectBase。
# 1. 类的主体
```csharp
using Battlehub;
using Battlehub.RTCommon;
using Battlehub.RTEditor;
using Battlehub.RTEditor.Views;
using System;

using UnityEngine;

public class TrussModel : ModelGameobjectBase
{
    [SerializeField, SerializeIgnore]
    private bool mIsingle;

    private IKTrussController mIKTruss;
    public IKTrussController Controller
    {
        get
        {
            return mIKTruss;
        }
    }

    /// <summary>
    /// 是否是单臂桁架
    /// </summary>
    public bool IsSingle
    {
        set
        {
            mIsingle = value;
        }
    }
    public override void Awake()
    {
        base.Awake();
        type = ModelType.Truss;//模型类型
        mIKTruss = GetComponent<IKTrussController>();
    }

    public override void Start()
    {
        //根据桁架类型，设置装配点可装配的模型类型
        if (mIsingle)
            assembleItemsType = new ModelType[] { ModelType.SuckerTruss };
        else
            assembleItemsType = new ModelType[] { ModelType.SuckerTruss, ModelType.SuckerTruss };

        base.Start();

        var lockAxes = gameObject.GetComponent<LockAxes>();
        if (lockAxes == null)
            lockAxes = gameObject.AddComponent<LockAxes>();

        lockAxes.RotationX = lockAxes.RotationZ = lockAxes.RotationFree = lockAxes.RotationScreen = true;
        lockAxes.ScaleX = lockAxes.ScaleY = lockAxes.ScaleZ = lockAxes.RectXY = lockAxes.RectXZ = lockAxes.RectYZ = true;
    }
    public override void OnDestroy()
    {
        base.OnDestroy();
    }
}

```