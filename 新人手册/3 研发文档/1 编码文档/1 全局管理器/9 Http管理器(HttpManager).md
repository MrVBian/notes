[TOC]

Http发送请求数据，接收返回数据，通过事件传递返回数据

# 1. 构造方法及单例
构造方法，私有，无参，初始化数据
```csharp
    private HttpManager()
    {
        mGetBuilder = new StringBuilder();
        mHttpClient = new HttpClient();

        mActionDic = new Dictionary<int, Action<HttpWebResponseDataBase>>();
    }
```
单例
```csharp
    private static HttpManager mInstance;
    public static HttpManager GetInstance()
    {
        if (mInstance == null)
            mInstance = new HttpManager();

        return mInstance;
    }
```
# 2. 类的成员
## 2.1 成员方法
```csharp
    /// <summary>
    /// 发送Get方式的请求
    /// </summary>
    /// <param name="url"></param>
    /// <param name="param"></param>
    /// <param name="action"></param>
    public void HttpGet(string url, string param, Action<HttpWebResponseDataBase> action)
    {
        int _key = GetKey();
        if (action != null)
        {
            mActionDic.Add(_key, action);
        }
        mGetBuilder.Clear();
        mGetBuilder.Append(url);
        if (param != null)
        {
            mGetBuilder.Append("?");
            mGetBuilder.Append(param);
        }

        HttpWebRequestMsg _data = new HttpWebRequestMsg(_key);
        _data.url = mGetBuilder.ToString();
        _data.type = HttpType.Get;
        mHttpClient.AddHttpReq(_data);
    }
```
```csharp
    /// <summary>
    /// 发送Post方式的请求
    /// </summary>
    /// <param name="reqdata"></param>
    /// <param name="url"></param>
    /// <param name="action"></param>
    public void HttpPost(HttpWebRequestDataBase reqdata, string url, Action<HttpWebResponseDataBase> action)
    {
        int _key = GetKey();
        if (action != null)
        {
            mActionDic.Add(_key, action);
        }

        mGetBuilder.Clear();
        mGetBuilder.Append(url);

        HttpWebRequestMsg _data = new HttpWebRequestMsg(_key);
        _data.url = mGetBuilder.ToString();
        _data.type = HttpType.Post;
        _data.data = reqdata;
        mHttpClient.AddHttpReq(_data);
    }
```
<center>支持Get和Post两种请求方式</center>

```csharp
    /// <summary>
    /// 创建请求唯一key值
    /// </summary>
    /// <returns></returns>
    private int GetKey()
    {
        mKey = 0;
        while (true)
        {
            if (!mActionDic.ContainsKey(mKey))
                break;

            mKey += 1;
        }
        return mKey;
    }
```
<center>创建消息唯一key值</center>

```csharp
    /// <summary>
    /// 实时检测消息是否有返回数据
    /// </summary>
    /// <param name="deltaTime"></param>
    public override void Update(float deltaTime)
    {
        if (TryGetResPonse(out mData))
        {
            if (mData.code == 200)
            {
                if (mActionDic.ContainsKey(mData.Key))
                {
                    mActionDic[mData.Key]?.Invoke(mData.data);
                    mActionDic.Remove(mData.Key);
                }
            }
            else
            {
                UnityEngine.Debug.LogError("http错误");
            }
        }
    }
    /// <summary>
    /// 有返回数据判断
    /// </summary>
    /// <param name="data"></param>
    /// <returns></returns>
    public bool TryGetResPonse(out HttpWebResponseMsg data)
    {
        if (mHttpClient.TryGetResponse(out data))
        {
            return true;
        }
        return false;
    }
```
<center>在Update方法,实时检测消息是否有返回数据,如有返回数据，根据key值，响应对应事件</center>