[TOC]

# 1. 构造方法及单例
构造方法，私有，无参，初始化
```csharp
    private MQTTManager()
    {
        robotMQTTData = new Dictionary<string, List<RobotMQTTData>>();
        manufactureMQTTData = new Dictionary<string, List<ManufactureMQTTData>>();
        topic = new Dictionary<string, bool>();
    }
```
单例
```csharp
    private static MQTTManager mInstance;
    public static MQTTManager GetInstance()
    {
        if (mInstance == null)
            mInstance = new MQTTManager();

        return mInstance;
    }
```