[TOC]


机器人模型类，继承ModelGameobjectBase。
# 1. 成员方法
## 1. 1 复写Awake方法
```csharp
    public override void Awake()
    {
        base.Awake();
        type = ModelType.Robot;

        mController = GetComponent<IKController>();
        var lockAxes = gameObject.GetComponent<LockAxes>();
        if (lockAxes == null)
        {
            lockAxes = gameObject.AddComponent<LockAxes>();
        }


        lockAxes.PositionY = lockAxes.RotationX = lockAxes.RotationZ = lockAxes.RotationFree = lockAxes.RotationScreen = true;
        lockAxes.ScaleX = lockAxes.ScaleY = lockAxes.ScaleZ = lockAxes.RectXY = lockAxes.RectXZ = lockAxes.RectYZ = true;
    }
```
设置模型类型为机器人类型(ModelType.Robot),添加限制Tranform各个参数的脚本（LockAxes）

## 1. 2 复写Start方法
```csharp
    public override void Start()
    {
        assembleItemsType = new ModelType[] { ModelType.SuckerRobot };

        base.Start();
        Transform Z01 = transform.Find("Z01");
        Transform X01 = transform.Find("X01");
        if (Z01 == null && X01 == null)
        {
            return;
        }
        Transform link1 = transform.Find("Link1");
        Transform link3 = transform.Find("Link1/Link2/Link3");

        if (link3 != null)
        {
            RobotRigidbody(link3);
        }

        Transform Z02 = transform.Find("Z02");
        if (Z02 != null)
        {
            RobotRigidbody(Z02, false, link3, false, -500f, -300f);
        }

        if (Z01 != null)
        {
            RobotRigidbody(Z01, false, Z02, true, -300f, -500f, null, -75f, 75f);
        }

        Transform link2 = transform.Find("Link1/Link2");
        if (link2 != null)
        {
            Transform X03 = transform.Find("X03");
            Transform X04 = transform.Find("X04");

            if (X03 != null)
            {
                RobotRigidbody(X03, false, link1);

                RobotRigidbody(X04, false, X03, true);
            }

            RobotRigidbody(link2, true, Z01, false, 0, 0, X04);
        }

        if (X01 != null)
        {
            RobotRigidbody(X01, false, link2);

            Transform X02 = transform.Find("X02");
            if (X02 != null)
            {
                RobotRigidbody(X02, false, X01, true);

                LinkOne(link1, X02);

            }
        }
    }
```
设置机器人可装配模型的类型，初始化铰链数据。

## 1. 3 铰链结构设置
```csharp
 void RobotRigidbody(Transform a, bool isKinematic = true, Transform connectedBody = null, bool useLimits = false, float X = 0f, float Y = 0f, Transform ace = null, float minn = 0, float maxx = 0)
    {
        if (ace != null)//双液压杆
        {
            Rigidbody rb = a.gameObject.GetComponent<Rigidbody>();
            rb.mass = 8f;
            rb.drag = 5f;
            rb.angularDrag = 1f;
            rb.useGravity = false;
            rb.isKinematic = isKinematic;
        }
        else//常规
        {
            Rigidbody rb = a.gameObject.GetComponent<Rigidbody>();
            if (rb == null)
            {
                rb = a.gameObject.AddComponent<Rigidbody>();
            }
            rb.mass = 8f;
            rb.drag = 5f;
            rb.angularDrag = 1f;
            rb.useGravity = false;
            rb.isKinematic = isKinematic;

        }

        if (connectedBody != null)
        {
            if (ace != null)//x04
            {
                HingeJoint hj2 = a.gameObject.GetComponent<HingeJoint>();//本体

                hj2.connectedBody = ace.GetComponent<Rigidbody>();
            }
            HingeJoint hj = a.GetComponent<HingeJoint>();
            if(hj==null)
                hj = a.gameObject.AddComponent<HingeJoint>();
            hj.connectedBody = connectedBody.GetComponent<Rigidbody>();
            hj.useLimits = useLimits;
            hj.limits = new JointLimits { min = minn, max = maxx };
            hj.anchor = new Vector3(0, 0, 0);
            hj.axis = new Vector3(0, 0, 1);
        }
        ConstantForce CF = a.GetComponent<ConstantForce>();
        if (CF == null)
        {
            CF = a.gameObject.AddComponent<ConstantForce>();
        }
        CF.relativeForce = new Vector3(X, Y, 0f);

    }

    void LinkOne(Transform ace, Transform bce)
    {

        Rigidbody rb = ace.gameObject.GetComponent<Rigidbody>();
        rb.mass = 10f;
        rb.drag = 10f;
        rb.angularDrag = 10f;

        rb.useGravity = false;

        rb.isKinematic = true;

        HingeJoint hj = ace.gameObject.GetComponent<HingeJoint>();
        hj.connectedBody = bce.GetComponent<Rigidbody>();
        hj.useLimits = false;
        hj.axis = new Vector3(0f, 0f, 1f);
    }
```