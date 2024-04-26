[TOC]


桁架机械臂移动，桁架机械臂拖拽，桁架机械臂示教走点。

# 1. 设置拖拽状态（SetDragState）
```csharp
    /// <summary>
    /// 设置桁架拖拽状态
    /// </summary>
    /// <param name="flag"></param>
    public void SetDragState(bool flag)
    {
        mDragFlag = flag;
        mSelectIndex = -1;
        for (int i = 0; i < mLength; i++)
        {
            mTarget[i].gameObject.SetActive(flag);
            mTargetEditor[i].MarkAsDestroyed = !flag;
        }
    }
```
# 2.桁架机械臂末端坐标显示（SetRobotDragInfo）
```csharp
    /// <summary>
    /// 设置桁架机械臂末端坐标显示
    /// </summary>
    /// <param name="obj"></param>
    /// <param name="id"></param>
    public void SetRobotDragInfo(GameObject obj,string id)
    {
        int _index = int.Parse(id)-1;

        if (mDragInfo == null)
        {
            mDragInfo = obj.transform;
            mDragTxt = mDragInfo.GetComponentInChildren<Text>();
            mDragTxt.rectTransform.anchoredPosition3D = new Vector3(0, -15, 0);
        }

        mDragInfo.SetParent(mTarget[_index]);
        mDragInfo.localPosition = Vector3.zero;

        float x = 0, y = 0, z = 0, rx = 0;
        GetTargetMoveById(id, ref x, ref y, ref z, ref rx);
        mDragTxt.text = "x:" + x + " y:" + y + " z:" + z + " w:" + rx;
    }
```
# 3. 拖拽桁架臂末端逻辑(SetTargetDrag)
```csharp
    /// <summary>
    /// 拖拽桁架臂末端
    /// </summary>
    private void SetTargetDrag()
    {
        if (mDragFlag&&!mMoveFlag)
        {
            if (mEditor.Selection.activeGameObject == null) return;
            
            if (Input.GetMouseButton(0))
            {
                if (link == null || link.Length != 9 || orientation == null || orientation.Length != 9) return;
                if (mSelectIndex == -1)
                {
                    for (int i = 0; i < mLength; i++)
                    {
                        if (mEditor.Selection.activeGameObject.name.Equals(mTarget[i].name))
                        {
                            mSelectIndex = i;
                            break;
                        }
                    }
                }
                if (mSelectIndex != -1)
                {
                    bool _positionflag = false;
                    if (mPos[mSelectIndex].x != mTarget[mSelectIndex].position.x)
                    {
                        float _x = mTarget[mSelectIndex].position.x - mPos[mSelectIndex].x;
                        _x = mMoveX[mSelectIndex].localPosition.x + _x - mInitOffset[mSelectIndex + 1].x;
                        _x = _x * mLinks[mSelectIndex * 4 + 1].transform.localScale.x / orientation[mSelectIndex * 4 + 1];
                        if (_x > mLimit_Upper[mSelectIndex, 0])
                        {
                            _x = mLimit_Upper[mSelectIndex, 0];
                        }
                        else if (_x < mLimit_Lower[mSelectIndex, 0])
                        {
                            _x = mLimit_Lower[mSelectIndex, 0];
                        }
                        if (trussType == TrussType.DoubleY)
                        {
                            link[5] = link[1] = _x;
                        }
                        else
                        {
                            link[mSelectIndex * 4 + 1] = _x;
                        }
                        
                        mMoveX[mSelectIndex].localPosition = new Vector3(orientation[mSelectIndex * 4 + 1] * _x / mLinks[mSelectIndex * 4 + 1].transform.localScale.x, mMoveX[mSelectIndex].localPosition.y, mMoveX[mSelectIndex].localPosition.z);
                        _positionflag = true;
                    }

                    if (mPos[mSelectIndex].y != mTarget[mSelectIndex].position.y)
                    {
                        float _y = mTarget[mSelectIndex].position.y - mPos[mSelectIndex].y;

                        _y = mMoveY[mSelectIndex].localPosition.y + _y - mInitOffset[mSelectIndex + 3].y;
                        _y = _y * mLinks[mSelectIndex * 4 + 3].transform.localScale.y / orientation[mSelectIndex * 4 + 3];
                        if (_y > mLimit_Upper[mSelectIndex, 2])
                        {
                            _y = mLimit_Upper[mSelectIndex, 2];
                        }
                        else if (_y < mLimit_Lower[mSelectIndex, 2])
                        {
                            _y = mLimit_Lower[mSelectIndex, 2];
                        }
                        link[mSelectIndex * 4 + 3] = _y;
                        mMoveY[mSelectIndex].localPosition = new Vector3(mMoveY[mSelectIndex].localPosition.x, orientation[mSelectIndex * 4 + 3] * _y / mLinks[mSelectIndex * 4 + 3].transform.localScale.y, mMoveY[mSelectIndex].localPosition.z);
                        _positionflag = true;
                    }

                    if (mPos[mSelectIndex].z != mTarget[mSelectIndex].position.z)
                    {
                        float _z = mTarget[mSelectIndex].position.z - mPos[mSelectIndex].z;

                        _z = mMoveZ[mSelectIndex].localPosition.z + _z - mInitOffset[mSelectIndex + 2].z;
                        _z = _z * mLinks[mSelectIndex * 4 + 2].transform.localScale.z / orientation[mSelectIndex * 4 + 2];
                        if (_z > mLimit_Upper[mSelectIndex, 1])
                        {
                            _z = mLimit_Upper[mSelectIndex, 1];
                        }
                        else if (_z < mLimit_Lower[mSelectIndex, 1])
                        {
                            _z = mLimit_Lower[mSelectIndex, 1];
                        }
                        link[mSelectIndex * 4 + 2] = _z;


                        mMoveZ[mSelectIndex].localPosition = new Vector3(mMoveZ[mSelectIndex].localPosition.x, mMoveZ[mSelectIndex].localPosition.y, orientation[mSelectIndex * 4 + 2] * _z / mLinks[mSelectIndex * 4 + 2].transform.localScale.z);
                        _positionflag = true;
                    }
                    if (_positionflag)
                    {
                        mPos[mSelectIndex] = mTarget[mSelectIndex].position;
                        if (mDragInfo != null && mDragInfo.parent != mTarget[mSelectIndex])
                        {
                            mDragInfo.SetParent(mTarget[mSelectIndex]);
                        }
                        SetLink();
                    }
                    
                    if (mAng[mSelectIndex] != mTargetEditor[mSelectIndex].LocalEuler.y)
                    {
                        float _y = mTargetEditor[mSelectIndex].LocalEuler.y - mAng[mSelectIndex] -mInitOffset[mSelectIndex + 4].y;
                        mCurrentAng[mSelectIndex] += _y;

                        _y = -(mCurrentAng[mSelectIndex] + 90) / orientation[mSelectIndex * 4 + 4];
                        
                        if (_y > mLimit_Upper[mSelectIndex, 3])
                        {
                            _y = mLimit_Upper[mSelectIndex, 3];

                        }
                        else if (_y < mLimit_Lower[mSelectIndex, 3])
                        {
                            _y = mLimit_Lower[mSelectIndex, 3];
                        }
                        link[mSelectIndex * 4 + 4] = _y;

                        mEnd[mSelectIndex].localEulerAngles = new Vector3(mEnd[mSelectIndex].localEulerAngles.x, orientation[mSelectIndex * 4 + 4] * -_y - 90f, mEnd[mSelectIndex].localEulerAngles.z);
                        mAng[mSelectIndex] = mTargetEditor[mSelectIndex].LocalEuler.y;
                        if (mDragInfo != null&&mDragInfo.parent!=this.transform)
                        {
                            mDragInfo.SetParent(this.transform);
                        }
                        SetLink();
                    }
                }
            }

            if (mSelectIndex != -1&&Input.GetMouseButtonUp(0))
            {
                mPos[mSelectIndex]= mEnd[mSelectIndex].position;
                mTarget[mSelectIndex].position = mPos[mSelectIndex];

                mAng[mSelectIndex] = mEnd[mSelectIndex].eulerAngles.y;
                mTarget[mSelectIndex].eulerAngles = mEnd[mSelectIndex].eulerAngles;

                mAng[mSelectIndex] = 0;
                mTarget[mSelectIndex].eulerAngles = Vector3.zero;
                mCurrentAng[mSelectIndex] = 0;

                mSelectIndex = -1;
            }

            if (mSelectIndex == -1)
            {
                for (int i = 0; i < mLength; i++)
                {
                    if (mPos[i] != mEnd[i].position)
                    {
                        mPos[i] = mEnd[i].position;
                        mTarget[i].position = mPos[i];
                    }
                }
            }

        }
    }
```
# 4. 示教走点
```csharp
 private void KeyCodeUpEvent(object[] args)
    {
        if (!Globals.ViewManager.SelectWindow.name.Equals(BuiltInWindowNames.Scene)) return;

        if (!(mDragFlag && !mMoveFlag) || mEditor.Selection.activeGameObject==null|| args.Length == 0 || !(args[0] is KeyCode)) return;

        KeyCode _keycode = (KeyCode)args[0];

        if (_keycode == KeyCode.Return)
        {
            for (int i = 0; i < mLength; i++)
            {
                if (mEditor.Selection.activeGameObject.name.Equals(mTarget[i].name))
                {
                    bool _flag = false;
                    int _index = i * 4;
                    for (int j = 1; j <= 4; j++)
                    {
                        if (j == 4)
                        {
                            if (mLinks[_index + j].transform.localEulerAngles != mInitVec[_index + j])
                            {
                                _flag = true;
                                break;
                            }
                        }
                        else
                        {
                            if (mLinks[_index + j].transform.localPosition != mInitVec[_index + j])
                            {
                                _flag = true;
                                break;
                            }
                        }
                    }
                    if (_flag)
                    {
                        mEditor.Selection.activeGameObject = null;
                        SetDragState(false);

                        mMoveFlag = true;

                        Vector3 _posX = mLinks[_index + 1].transform.localPosition;
                        mLinks[_index + 1].transform.localPosition = new Vector3(mInitVec[_index + 1].x, _posX.y, _posX.z);

                        Vector3 _posZ = mLinks[_index + 2].transform.localPosition;
                        mLinks[_index + 2].transform.localPosition = new Vector3(_posZ.x, _posZ.y, mInitVec[_index + 2].z);

                        Vector3 _posY = mLinks[_index + 3].transform.localPosition;

                        bool _moveYflag = false;
                        if (mInitVec[_index + 3].y > _posY.y)
                        {
                            _moveYflag = false;
                        }
                        else
                        {
                            _moveYflag = true;
                        }

                        mLinks[_index + 3].transform.localPosition = new Vector3(_posY.x, mInitVec[_index + 3].y, _posY.z);

                        Vector3 _angleY = mLinks[_index + 4].transform.localEulerAngles;
                        mLinks[_index + 4].transform.localEulerAngles = new Vector3(_angleY.x, mInitVec[_index + 4].y, _angleY.z);

                        mStep = 0;

                        if (_moveYflag)
                        {
                            mTweeners.Add(mLinks[_index + 3].transform.DOLocalMoveY(_posY.y, 0.5f).SetEase(Ease.Linear)
                                  .SetSpeedBased().OnComplete(() => StepComplete(_index, _posX, _posZ, _angleY)));
                        }
                        else
                        {
                            mTweeners.Add(mLinks[_index + 1].transform.DOLocalMoveX(_posX.x, 0.5f).SetEase(Ease.Linear)
                                .SetSpeedBased().OnComplete(() => StepComplete(_index, _posY)));
                            mTweeners.Add(mLinks[_index + 2].transform.DOLocalMoveZ(_posZ.z, 0.5f).SetEase(Ease.Linear)
                                .SetSpeedBased().OnComplete(() => StepComplete(_index, _posY)));
                            mTweeners.Add(mLinks[_index + 4].transform.DOLocalRotate(_angleY, 30f).SetEase(Ease.Linear)
                                .SetSpeedBased().OnComplete(() => StepComplete(_index, _posY)));
                        }
                    }
                    break;
                }
            }
        }
    }

    private void StepComplete(int index, Vector3 posX, Vector3 posZ, Vector3 angleY)
    {
        mStep += 1;
        if (mStep == 1)
        {
            mTweeners.Add(mLinks[index + 1].transform.DOLocalMoveX(posX.x, 0.5f).SetEase(Ease.Linear)
                 .SetSpeedBased().OnComplete(() => StepComplete(index, posX, posZ, angleY)));
            mTweeners.Add(mLinks[index + 2].transform.DOLocalMoveZ(posZ.z, 0.5f).SetEase(Ease.Linear)
                .SetSpeedBased().OnComplete(() => StepComplete(index, posX, posZ, angleY)));
            mTweeners.Add(mLinks[index + 4].transform.DOLocalRotate(angleY, 30f).SetEase(Ease.Linear)
                .SetSpeedBased().OnComplete(() => StepComplete(index, posX, posZ, angleY)));
        }
        else if (mStep == 4)
        {
            mStep = 0;
            mMoveFlag = false;
            mTweeners.Clear();
            SetInitVec();
            SetDragState(true);
            
        }
    }

    private void StepComplete(int index,Vector3 vec)
    {
        mStep += 1;
        if (mStep == 3)
        {
            mTweeners.Add(mLinks[index + 3].transform.DOLocalMoveY(vec.y, 0.5f).SetEase(Ease.Linear)
                .SetSpeedBased().OnComplete(() => StepComplete(index, vec)));
        }
        else if(mStep==4)
        {
            mStep = 0;
            mMoveFlag = false;
            mTweeners.Clear();
            SetInitVec();
            SetDragState(true);
        }
    }
```