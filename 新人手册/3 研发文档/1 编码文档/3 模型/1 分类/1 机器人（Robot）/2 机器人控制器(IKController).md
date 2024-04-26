[TOC]


处理机器人逆解逻辑，机器人数据虚实对应，机器人示教走点。
# 1. MPCSDK
## 1.1 DLL库
```csharp
    public class SBTMpc
    {
        public SBTMpc();

        ~SBTMpc();

        public bool create(string config, Encoding encoding);
        public SingleListD forceControlBuilderTest(List<double> TargetPoint_previous_in, List<double> TargetPoint_current_in, List<double> TargetPoint_next_in, List<double> CompliantPoint_previous_in, List<double> CompliantPoint_current_in, List<double> ForceVec_desired_previous_in, List<double> ForceVec_desired_current_in, List<double> ForceVec_measured_previous_in, List<double> ForceVec_measured_current_in, double SamplingTime);
        public SingleListD forceControlBuilderTestChangeFrame(List<double> TargetPoint_previous);
        public MPCGetJointLimitsResult getJointLimits();
        public MPCGetJointSignResult getJointSigns();
        public MPCReal2TheoreticalResult real2Theoretical(List<double> realjp);
        public MPCFKUnityResult SolveFKUnity(List<double> jointPositionsUnityWorld);
        public MPCIKResult solveIK(List<double> EEtarget_position_xyz_UnityWorld, List<double> EEtarget_quaternion_wxyz_UnityWorld, List<double> RobotBase_position_xyz_UnityWorld, List<double> RobotBase_quaternion_wxyz_UnityWorld, List<double> UserInput_TargetPosition_wrtRobotBase, List<double> UserInput_TargetEulerAngles_wrtRobotBase, List<double> JointPositions_current, string priorities, int max, double pos, double ori);
        public MPCTheoretical2RealResult theoretical2Real(List<double> theoretical);
    }
```
## 1.2 初始化
```csharp
        mMpc = new SBTMpc();
        Encoding _encoding = Encoding.GetEncoding("GBK");
        mCreateFlag = mMpc.create(UtilsLogic.GetStringByEncoding(path, _encoding), _encoding);
        mJointSigns = mMpc.getJointSigns().data;
        mLowerLimit = mMpc.getJointLimits().lower;
        mUpperLimit = mMpc.getJointLimits().upper;
```
<center>初始化SDK，获得关节极限值和标签值。</center>
## 1. 3 solveIK逆解
```csharp
                xyz.Add(target.position.x);
                xyz.Add(target.position.y - mOffset);
                xyz.Add(target.position.z);

                qxyz.Add(target.rotation.w);
                qxyz.Add(target.rotation.x);
                qxyz.Add(target.rotation.y);
                qxyz.Add(target.rotation.z);

                bxyz.Add(transform.position.x);
                bxyz.Add(transform.position.y);
                bxyz.Add(transform.position.z);

                bqxyz.Add(transform.rotation.w);
                bqxyz.Add(transform.rotation.x);
                bqxyz.Add(transform.rotation.y);
                bqxyz.Add(transform.rotation.z);

                jc.AddRange(mLastLinksAngle);

                mIKRet = mMpc.solveIK(xyz, qxyz, bxyz, bqxyz, userInputPos, userInputAng, jc, priorities, 20, 1e-3, 1e-3);
```
根据法兰中心点的位置和角度，逆解获得机器人关节的角度。

```csharp
            jc.AddRange(mLastLinksAngle);

            userInputPos.Add(pos.x);
            userInputPos.Add(pos.y);
            userInputPos.Add(pos.z);

            userInputAng.Add(ang.x);
            userInputAng.Add(ang.y);
            userInputAng.Add(ang.z);

            mIKRet = mMpc.solveIK(xyz, qxyz, bxyz, bqxyz, userInputPos, userInputAng, jc, "P<O", 100, 1e-3, 1e-3);
```
根据用户输入的真实坐标和角度，逆解获得机器人关节的角度。

# 2. 公共方法
## 2.1  SetShowData
```csharp
    /// <summary>
    /// 设置用户显示坐标和角度
    /// </summary>
    /// <param name="datas"></param>
    public void SetShowData(double[] datas)
    {
        mSingleData.Clear();
        mSingleData.AddRange(datas);
        MPCFKUnityResult _res = mMpc.SolveFKUnity(mSingleData);
        mShowData.Clear();
        mShowData.AddRange(_res.realRobotPosition);
        mShowData.AddRange(_res.realRobotRotation);

        double[] _real = GetJointsEulerAngleForShow();
        link1 = mLink1 = (float)Math.Round(_real[0], AppConst.digits);
        link2 = mLink2 = (float)Math.Round(_real[1], AppConst.digits);
        link3 = mLink3 = (float)Math.Round(_real[2], AppConst.digits);
        link4 = mLink4 = (float)Math.Round(_real[3], AppConst.digits);
        link5 = mLink5 = (float)Math.Round(_real[4], AppConst.digits);
        link6 = mLink6 = (float)Math.Round(_real[5], AppConst.digits);
    }
```
## 2. 2 GetJointsEulerAngleForShow
```csharp
    /// <summary>
    /// 获得关节的角度
    /// </summary>
    /// <param name="cfg"></param>
    /// <returns></returns>
    public double[] GetJointsEulerAngleForShow()
    {
        if (!mCreateFlag) return null;
        List<double> _list = GetLinksLocalAngle();
        double[] real = mMpc.theoretical2Real(_list).real;
        return real;
    }
```

## 2. 3 SetJointTargetValue
```csharp
    /// <summary>
    /// 根据用户输入关节数据和机器人的类型，设置机器人的姿态
    /// </summary>
    /// <param name="mAngList"></param>
    /// <param name="value"></param>
    /// <param name="index"></param>
    /// <param name="cfg"></param>
    public void SetJointTargetValue(List<float> mAngList, double value, int index)
    {
        if (!mCreateFlag || mDebuggingFlag) return;
        List<double> _inputlist = new List<double>();
        for (int i = 0; i < mAngList.Count; i++)
        {
            if (i == mAngList.Count - 1) continue;

            if (i == index)
            {
                mAngList[i] = (float)value;
                _inputlist.Add(value);

            }
            else
            {
                _inputlist.Add((double)mAngList[i]);
            }
        }

        double[] _theoretical = mMpc.real2Theoretical(_inputlist).theoretical;
        if (_theoretical == null || _theoretical.Length != 6) return;
        for (int i = 0; i < _theoretical.Length; i++)
        {
            SetLink(i, (float)_theoretical[i]);
        }

        SetShowData(_theoretical);
        SeTargetPosAndAng(mEnd);

        RobotDragFlagAction?.Invoke();
        ShowDragTxt();
    }
```
## 2. 4 获取关节极限值
```csharp
    /// <summary>
    /// 获得关节角度最小值
    /// </summary>
    /// <param name="index"></param>
    /// <returns></returns>
    public double GetJointLimitLower(int index)
    {
        if (mLowerLimit == null || mLowerLimit.Length != 6)
            return 0;
        return mLowerLimit[index];
    }
    /// <summary>
    /// 获得关节角度最大值
    /// </summary>
    /// <param name="index"></param>
    /// <returns></returns>
    public double GetJointLimitUppder(int index)
    {
        if (mUpperLimit == null || mUpperLimit.Length != 6)
            return 0;

        return mUpperLimit[index];
    }
```
## 2. 5示教走点
```csharp
    /// 示教走点
    /// </summary>
    /// <param name="moveLinks"></param>
    /// <param name="end"></param>
    /// <param name="isMotion"></param>
    public void DebuggingMove(List<double> moveLinks, List<double> end,bool isMotion)
    {
        if (!mDebuggingFlag) return;
        bool _arrived = true;
        if (isMotion)
        {
            if (moveLinks.Count != end.Count) return;

            for (int i = 0; i < moveLinks.Count; i++)
            {
                if (Math.Abs(moveLinks[i] - end[i]) > 1f)
                {
                    _arrived = false;
                    break;
                }
            }
        }
        if (!_arrived)
        {
            
            for (int i = 0; i < moveLinks.Count; i++)
            {
                SetLink(i, (float)moveLinks[i]);
                SetShowData(moveLinks.ToArray());
            }
        }
        else
        {
            for (int i = 0; i < end.Count; i++)
            {
                SetLink(i, (float)end[i]);
            }
            SetShowData(end.ToArray());
            mDebuggingFlag = false;
            mRobotProgrammingState.SetRobotArrived(this);
            link = mTempLinks;
            Debug.Log("Arrived----->");
        }

        SeTargetPosAndAng(mEE_Frame);

        RobotDragFlagAction?.Invoke();
        ShowDragTxt();

        if (mRobotGripperEnd != null)
            SetLine(mRobotGripperEnd.position);
        else
            SetLine(mPos);
    }
```