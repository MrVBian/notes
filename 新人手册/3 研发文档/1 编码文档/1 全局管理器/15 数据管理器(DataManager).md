[TOC]

# 1. 构造方法及单例
构造方法，私有，无参,初始化
```csharp
    private DataManager()
    {
        map = new Dictionary<GameObject, string>();
        objectMap = new Dictionary<string, GameObject>();
        cameraDataMap = new Dictionary<string, CameraCapture>();
        uuidMap = new Dictionary<string, bool>();
    }
```
单例
```csharp
    private static DataManager mInstance;
    public static DataManager GetInstance()
    {
        if (mInstance == null)
            mInstance = new DataManager();

        return mInstance;
    }
```