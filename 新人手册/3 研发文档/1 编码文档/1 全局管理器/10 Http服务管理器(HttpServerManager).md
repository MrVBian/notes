[TOC]

将软件作为接收http请求的服务端，根据请求内容，创建返回数据，向请求客户端发送。

# 1. 构造方法及单例
构造方法，私有，无参
```csharp
    private HttpServerManager()
    {
    }
```
单例
```csharp
    private static HttpServerManager mInstance;
    public static HttpServerManager GetInstance()
    {
        if (mInstance == null)
            mInstance = new HttpServerManager();

        return mInstance;
    }
```
# 2. 类的成员
## 2.1 成员方法
```csharp
    /// <summary>
    /// 初始化
    /// </summary>
    public override void Init()
    {
        mClient = new HttpServerClient();
        mModule = new HttpServerModule();
        mClient.Connect();
    }
```
<center>初始化Http服务，建立服务连接</center>

```csharp
        /// <summary>
    /// 监听接收请求信息，根据请求内容，创建回传数据
    /// </summary>
    /// <param name="deltaTime"></param>
    public override void Update(float deltaTime)
    {
        if (mClient.GetTryGetResponse(out resdata))
        {
            mModule.HandleHttpServerReceiveData(resdata, HandleHttpServerReceiveDataComplete);
        }
    }
    /// <summary>
    /// 将回传数据加入到队列，等待发送
    /// </summary>
    /// <param name="data"></param>
    public void HandleHttpServerReceiveDataComplete(HttpServerResponseMsg data)
    {
        mClient.AddHttpResponse(data);
    }
```
<center>监听接收请求信息，根据请求内容，创建回传数据，创建完成后，加入消息答复队列，等待发送。</center>