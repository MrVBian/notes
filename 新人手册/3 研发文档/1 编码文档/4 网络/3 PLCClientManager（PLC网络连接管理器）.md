### 1. AddIoTSingalData
添加PLC客户端连接信息，用于创建PLC连接。
```csharp
    public void AddIoTSingalData(PLCSignalData data)
    {
        if (!mPLCSignalDataList.Contains(data))
        {
            Transform _trans = data.transform;
            if (mIoTObj == null)
            {
                if (_trans.parent != null)
                {
                    mIoTObj = _trans.parent.gameObject;
                }
                else
                {
                    mIoTObj = new GameObject("IoTSingalData");
                    mIoTObj.AddComponent<GameObjectSaveTag>().flag = true;
                }
            }

            _trans.SetParent(mIoTObj.transform);
            data.addFlag = true;
            mPLCSignalDataList.Add(data);
            data.PLCName = "PLC设备" + mPLCSignalDataList.Count;
            data.gameObject.name= "PLC设备" + mPLCSignalDataList.Count;

            //data.outputChanged += (index, f) => PLCSignalDataOutputChanged(data,index, f);

            data.inputChanged += (index, v) => PLCSignalDataInputChanged(data, index, v);
        }
    }
```
### 2. GetOrCreateClient
根据协议类型，ip地址，端口号，获得已创建的PLC客户连接。
```csharp
    /// <summary>
    /// 创建PLC连接客户端
    /// </summary>
    /// <param name="deviceVersion"></param>
    /// <param name="ip"></param>
    /// <param name="port"></param>
    /// <returns></returns>
    public IIoTClientCommon GetOrCreateClient(EthernetDeviceVersion deviceVersion, string ip, int port)
    {
        IIoTClientCommon _client = GetClient(deviceVersion, ip, port);

        if (_client == null)
        {
            PLCClientData _data = new PLCClientData();
            _data.deviceVersion = deviceVersion;
            _data.ip = ip;
            _data.port = port;
            _client = IoTClientFactory.CreateClient2EthernetDevice(deviceVersion, ip, port);
            mClientDic.Add(_data, _client);
        }
        return _client;
    }
```
### 3. OpenReadClient
根据客户端连接信息，创建多个多线程，开启数据读取客户端连接。
```csharp
    /// <summary>
    /// 开启数据读取客户端连接
    /// </summary>
    public void OpenReadClient()
    {
        mReadPLCSignalDataList.Clear();
        if (mPLCSignalDataList.Count != 0)
        {
            mPLCSignalDataList.ForEach(
                (v) =>
                {
                    //有可能要修改
/*                    if (v.IoDatasDic.Count != 0)
                    {
                        mReadPLCSignalDataList.Add(v);
                    }*/
                    if (v.inputAddressDataForUI.Length != 0)
                    {
                        mReadPLCSignalDataList.Add(v);
                    }
                }
            );
        }
        //Debug.Log("OpenReadClient--->" + mReadPLCSignalDataList.Count);
        if (mReadPLCSignalDataList.Count != 0)
        {
            mCancelTokenSource = new CancellationTokenSource();
            mTask = Task.Run(() => 
            {
                mReadPLCSignalDataList.ForEach((v) =>
                {
                    IIoTClientCommon _client = GetOrCreateClient(v.deviceVersion, v.ip, v.port);
                    _client.Open();

                    if (mReadClientDic.ContainsKey(_client))
                    {
                        mReadClientDic[_client].Add(v);
                    }
                    else
                    {
                        List<PLCSignalData> _list = new List<PLCSignalData>();
                        _list.Add(v);
                        mReadClientDic.Add(_client, _list);
                    }
                });
            }).
           ContinueWith((v)=> 
            {
                Debug.Log("mReadClientDic---->" + mReadClientDic.Count);
                mIsConnect = true;
                foreach (var key in mReadClientDic.Keys)
                {
                    CancellationTokenSource  _cancelTokenSource = new CancellationTokenSource();
                    Task _task = Task.Factory.StartNew(()=>ReadAysn(key, mReadClientDic[key], _cancelTokenSource));
                    mTaskDic.Add(_cancelTokenSource, _task);
/*                    mTokenList.Add(_cancelTokenSource);
                    mTaskList.Add(_task);*/
                }
            });
        }
    }
```
### 3. Read
将从服务端读取的数据，根据协议类型和指定数据类型，进行处理。
```csharp
 private void Read(PLCSignalData data)
    {
        IIoTClientCommon _client = GetOrCreateClient(data.deviceVersion, data.ip, data.port);
        if (_client == null || !_client.IsConnected) return;

        if (data.inputAddress == null)
        {
            data.SetInputAddress();
        }
        var _list = new Dictionary<string, DataTypeEnum>();
        DataTypeEnum _type;
        for (int i = 0; i < data.inputAddress.Length; i++)
        {
            string _value = data.inputAddress[i];
            if (string.IsNullOrEmpty(_value)) continue;
            OData _odata = null;
            _type = DataTypeEnum.None;
            foreach (var odata in data.oDatasDic.Keys)
            {
                if (odata.startint == i)
                //if (odata.plcAddress == i)
                {
                    _odata = odata;
                    break;
                }
            }

            if (_odata != null)
            {
                if (_odata.datatype == DataTypeEn.tfloat)
                {
                    _type = DataTypeEnum.Float;
                }
                else if (_odata.datatype == DataTypeEn.tshort)
                {
                    _type = DataTypeEnum.Int16;
                }
                else if (_odata.datatype == DataTypeEn.tint)
                {
                    _type = DataTypeEnum.Int32;
                }
                else if(_odata.datatype == DataTypeEn.tbool)
                {
                    switch (data.deviceVersion)
                    {
                        case EthernetDeviceVersion.ModbusTcp:
                            _type = DataTypeEnum.UInt16;
                            break;
                        case EthernetDeviceVersion.Siemens_S7_200:
                        case EthernetDeviceVersion.Siemens_S7_200Smart:
                        case EthernetDeviceVersion.Siemens_S7_300:
                        case EthernetDeviceVersion.Siemens_S7_400:
                        case EthernetDeviceVersion.Siemens_S7_1200:
                        case EthernetDeviceVersion.Siemens_S7_1500:
                            _type = DataTypeEnum.Byte;
                            break;
                    }
                }
            }
            if (_type == DataTypeEnum.None) continue;
            if (!_list.ContainsKey(_value))
            {
                _list.Add(_value, _type);
            }
        }
        var _result = _client.BatchRead(_list, _list.Count);
        if (_result != null)
        {
            if (_result.IsSucceed)
            {
                for (int i = 0; i < data.inputAddress.Length; i++)
                {
                    if (string.IsNullOrEmpty(data.inputAddress[i])) continue;
                    object _value = null;
                    foreach (var key in _result.Value.Keys)
                    {
                        if (data.inputAddress[i].Equals(key))
                        {
                            _value = _result.Value[key];
                            break;
                        }
                    }
                    if (_value == null) continue;
                    OData _odata = null;
                    foreach (var odata in data.oDatasDic.Keys)
                    {
                        if (odata.startint == i)
                        //if (odata.plcAddress == i)
                        {
                            _odata = odata;
                            break;
                        }
                    }
                    if (_odata == null) continue;
                    if (_odata.datatype == DataTypeEn.tfloat)
                    {
                        float _tempfloat = Convert.ToSingle(_value);
                        byte[] _floatBytes = BitConverter.GetBytes(_tempfloat);
                        string _binaryString = string.Join("", _floatBytes.Select(b => Convert.ToString(b, 2).PadLeft(8, '0')));
                        Debug.Log("address--->" + data.inputAddress[i] + "Value--->" + _value + "ushort:" + _tempfloat + "_binaryString:" + _binaryString);
                        for (int j = 0; j < _binaryString.Length; j++)
                        {
                            bool _v = _binaryString[j].Equals('0') ? false : true;
                            int _index = _odata.startint + _binaryString.Length - 1 - j;
                            if (data.Output[_index] != _v)
                            {
                                data.Output[_index] = _v;
                            }
                        }
                    }
                    else if (_odata.datatype == DataTypeEn.tshort)
                    {
                        short _tempshort = Convert.ToInt16(_value);
                        byte[] _shortBytes = BitConverter.GetBytes(_tempshort);
                        string _binaryString = string.Join("", _shortBytes.Select(b => Convert.ToString(b, 2).PadLeft(8, '0')));
                        Debug.Log("address--->" + data.inputAddress[i] + "Value--->" + _value + "short:" + _tempshort + "_binaryString:" + _binaryString);
                        for (int j = 0; j < _binaryString.Length; j++)
                        {
                            bool _v = _binaryString[j].Equals('0') ? false : true;
                            int _index = _odata.startint + _binaryString.Length - 1 - j;
                            if (data.Output[_index] != _v)
                            {
                                data.Output[_index] = _v;
                            }
                        }
                    }
                    else if (_odata.datatype == DataTypeEn.tint)
                    {
                        int _tempint = Convert.ToInt32(_value);
                        byte[] _intBytes = BitConverter.GetBytes(_tempint);
                        string _binaryString = string.Join("", _intBytes.Select(b => Convert.ToString(b, 2).PadLeft(8, '0')));
                        Debug.Log("address--->" + data.inputAddress[i] + "Value--->" + _value + "int:" + _tempint + "_binaryString:" + _binaryString);
                        for (int j = 0; j < _binaryString.Length; j++)
                        {
                            bool _v = _binaryString[j].Equals('0') ? false : true;
                            int _index = _odata.startint + _binaryString.Length - 1 - j;
                            if (data.Output[_index] != _v)
                            {
                                data.Output[_index] = _v;
                            }
                        }
                    }
                    else if(_odata.datatype == DataTypeEn.tbool)
                    {
                        int _index = 0;
                        if (string.IsNullOrEmpty(data.inputAddressDataForUI[i].bitAddress) || !int.TryParse(data.inputAddressDataForUI[i].bitAddress, out _index))
                        {
                            _index = 0;
                        }
                        int _mask = 1 << _index;
                        switch (data.deviceVersion)
                        {
                            case EthernetDeviceVersion.ModbusTcp:
                                ushort _tempushort = Convert.ToUInt16(_value);
                                _tempushort = (ushort)(_tempushort & _mask);
                                bool _tempvalue = _tempushort != 0;
                                Debug.Log("address--->" + data.inputAddress[i] + "Value--->" + _value + "ushort:" + _tempushort);
                                if (_odata != null)
                                {
                                    if (_tempvalue != data.Output[_odata.startint])
                                    {
                                        data.Output[_odata.startint] = _tempvalue;
                                    }
                                }
                                break;
                            case EthernetDeviceVersion.Siemens_S7_200:
                            case EthernetDeviceVersion.Siemens_S7_200Smart:
                            case EthernetDeviceVersion.Siemens_S7_300:
                            case EthernetDeviceVersion.Siemens_S7_400:
                            case EthernetDeviceVersion.Siemens_S7_1200:
                            case EthernetDeviceVersion.Siemens_S7_1500:
                                byte _tempbyte = Convert.ToByte(_value);
                                _tempbyte = (byte)(_tempbyte & _mask);
                                bool _temp = _tempbyte != 0;
                                Debug.Log("address--->" + data.inputAddress[i] + "Value--->" + _value + "byte:" + _temp);
                                if (_odata != null)
                                {
                                    if (_temp != data.Output[_odata.startint])
                                    {
                                        data.Output[_odata.startint] = _temp;
                                    }
                                }
                                break;
                        }
                    }
                }
            }
            else
            {
                Debug.Log("ErrCode--->" + _result.ErrCode + "||" + _result.Err + "deviceVersion" + data.deviceVersion);
            }
        }
    }
```
### 4. Write
将Speedbuilder软件监听到数据，根据协议类型和指定数据类型，转换数据传给服务端。
```csharp
    private void Write(PLCSignalData data, int index)
    {
        IIoTClientCommon _client = GetOrCreateClient(data.deviceVersion, data.ip, data.port);

        if (data.outputAddress == null)
        {
            data.SetOutputAddress();
        }
        IoDta _iodta = null;
        foreach (var iodta in data.IoDatasDic.Keys)
        {
            if (iodta.startint == index)
            {
                _iodta = iodta;
                break;
            }
        }
        if (_iodta == null)
            return;
        string _address = "";
        PLCAddressData _addressdata = null;
        for (int i = 0; i < data.outputAddress.Length; i++)
        {
            if (_iodta.startint == i)
            //if (_iodta.plcAddress == i)
            {
                _address = data.outputAddress[i];
                _addressdata = data.outputAddressDataForUI[i];
                break;
            }
        }
        if (string.IsNullOrEmpty(_address)) return;
        lock (_client)
        {
            bool _flag = true;
            if (!_client.IsConnected)
            {
                _flag = false;
                _client.Open();
            }
            DataTypeEnum _type = DataTypeEnum.None;
            Result _result = null;
            if (_iodta.datatype == DataTypeEn.tfloat)
            {
                _type = DataTypeEnum.Float;
                string _binaryString = "";
                int _length = 32;
                for (int i = index + _length - 1; i >= index; i--)
                {
                    _binaryString += data.Input[i] ? "1" : "0";
                }
                float _v = BinaryToFloat32(_binaryString);
                /*_v = BinaryToFloat32("01111001111010011111011001000010");*/
                Debug.Log("float-->" + _v);
                _result = _client.Write(_address, _v, _type);
            }
            else if (_iodta.datatype == DataTypeEn.tshort)
            {
                _type = DataTypeEnum.Int16;
                string _binaryString = "";
                int _length = 16;
                for (int i = index + _length - 1; i >= index; i--)
                {
                    _binaryString += data.Input[i] ? "1" : "0";
                }
                int _v = BinaryToInt16(_binaryString);
                Debug.Log("int-->" + _v);
                _result = _client.Write(_address, _v, _type);
            }
            else if (_iodta.datatype == DataTypeEn.tint)
            {
                _type = DataTypeEnum.Int32;
                string _binaryString = "";
                int _length = 32;
                for (int i = index + _length - 1; i >= index; i--)
                {
                    _binaryString += data.Input[i] ? "1" : "0";
                }
                int _v = BinaryToInt32(_binaryString);
                Debug.Log("int-->" + _v);
                _result = _client.Write(_address, _v, _type);
            }
            else if(_iodta.datatype == DataTypeEn.tbool)
            {
                string _readadd = "";
                switch (data.deviceVersion)
                {
                    case EthernetDeviceVersion.ModbusTcp:
                        _type = DataTypeEnum.UInt16;
                        ModbusTCPSignalData _modbus = data as ModbusTCPSignalData;
                        _readadd = _modbus.stationNumber + "," + _addressdata.address + "," + _modbus.Read;
                        break;
                    case EthernetDeviceVersion.Siemens_S7_200:
                    case EthernetDeviceVersion.Siemens_S7_200Smart:
                    case EthernetDeviceVersion.Siemens_S7_300:
                    case EthernetDeviceVersion.Siemens_S7_400:
                    case EthernetDeviceVersion.Siemens_S7_1200:
                    case EthernetDeviceVersion.Siemens_S7_1500:
                        _type = DataTypeEnum.Byte;
                        _readadd = _address;
                        break;
                }
                if (!string.IsNullOrEmpty(_readadd))
                {
                    Debug.Log("readadd--->" + _readadd);
                    var _list = new Dictionary<string, DataTypeEnum>();
                    _list.Add(_readadd, _type);
                    var _readresult = _client.BatchRead(_list, _list.Count);
                    int _index = 0;
                    if (string.IsNullOrEmpty(_addressdata.bitAddress) || !int.TryParse(_addressdata.bitAddress, out _index))
                    {
                        _index = 0;
                    }
                    int _mask = 1 << _index;
                    if (_readresult.IsSucceed)
                    {
                        if (_readresult.Value.ContainsKey(_readadd))
                        {
                            object _readvalue = _readresult.Value[_readadd];
                            switch (data.deviceVersion)
                            {
                                case EthernetDeviceVersion.ModbusTcp:
                                    ushort _tempushort = Convert.ToUInt16(_readvalue);
                                    if (data.Input[index])
                                    {
                                        _tempushort = (ushort)(_tempushort | _mask);
                                    }
                                    else
                                    {
                                        _tempushort = (ushort)(_tempushort & ~_mask);
                                    }
                                    _result = _client.Write(_address, _tempushort, DataTypeEnum.UInt16);
                                    Debug.Log("_tempushort-->" + _tempushort);

                                    break;
                                case EthernetDeviceVersion.Siemens_S7_200:
                                case EthernetDeviceVersion.Siemens_S7_200Smart:
                                case EthernetDeviceVersion.Siemens_S7_300:
                                case EthernetDeviceVersion.Siemens_S7_400:
                                case EthernetDeviceVersion.Siemens_S7_1200:
                                case EthernetDeviceVersion.Siemens_S7_1500:
                                    byte _tempbyte = Convert.ToByte(_readvalue);
                                    if (data.Input[index])
                                    {
                                        _tempbyte = (byte)(_tempbyte | _mask);
                                    }
                                    else
                                    {
                                        _tempbyte = (byte)(_tempbyte & ~_mask);
                                    }
                                    _result = _client.Write(_address, _tempbyte, DataTypeEnum.Byte);
                                    Debug.Log("_tempbyte-->" + _tempbyte);
                                    break;
                            }
                        }
                    }
                }
            }
            if (_result != null && !_result.IsSucceed)
                Debug.Log("ConnectionInfo:" + _client.ConnectionInfo + "ErrCode:" + _result.ErrCode);

            if (!_flag)
            {
                _client.Close();
            }
        }
    }
```