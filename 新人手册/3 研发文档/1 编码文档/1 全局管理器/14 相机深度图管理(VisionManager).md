[TOC]

# 1. 构造方法及单例
构造方法，私有，无参
```csharp
    private VisionManager()
    {
    }
```
单例
```csharp
    private static VisionManager mInstance;

    public static VisionManager GetInstance()
    {
        if (mInstance == null)
            mInstance = new VisionManager();

        return mInstance;
    }
```