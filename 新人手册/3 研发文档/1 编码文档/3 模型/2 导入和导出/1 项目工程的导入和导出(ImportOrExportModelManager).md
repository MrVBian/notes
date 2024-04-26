[TOC]


工程项目自定义工程文件和模型文件的导入和导出。

# 1. GameObjectSaveTag
```csharp
public class GameObjectSaveTag : MonoBehaviour
{
    public bool flag;//是否需要导出标签
    [System.NonSerialized]
    public string GameObjectName;
    [System.NonSerialized]
    public string Path;
}
```
在场景中的模型，必须添加该脚本，在导出时，会根据flag的值，进行导出。

# 1. SaveProjectData
```csharp
/// <summary>
/// 工程项目场景数据
/// </summary>
[Serializable]
public class SaveProjectData 
{
    public string AppVersion;

    public List<SceneModelData> datalist = new List<SceneModelData>();
}
/// <summary>
/// 工程项目场景模型数据
/// </summary>
[Serializable]
public class SceneModelData
{
    public string name;

    public string GUID;
    /*    [JsonConverter(typeof(VectorConverter))]
        public Vector3 position;
        [JsonConverter(typeof(VectorConverter))]
        public Vector3 eulerangle;
        [JsonConverter(typeof(VectorConverter))]
        public Vector3 scale;*/

    public Vector3 position;

    public Vector3 eulerangle;

    public Vector3 scale;

    public string JsonModelGuid;

    public TransformParent parent;
}
/// <summary>
/// 工程项目场景模型父节点数据
/// </summary>
[Serializable]
public class TransformParent
{
    public string GUID;

    public string path;
}
```
<center>自定义工程文件数据</center>

# 3. ExportProjectAction导出工程
```csharp
public Action ExportProjectAction;
```
```csharp
ExportProjectAction += ExportProjectEvent;
```
```csharp
    private void ExportProjectEvent()
    {
        if (mFlag) return;
        Globals.TimerManager.Regist((v) => StandaloneFileBrowser.OpenFolderPanelAsync(mLocalization.GetString("ID_RTEditor_ExportProject"), null, false, ExportProjectCallback), 0.2f, 1);

        //Globals.TimerManager.Regist((v) => SavePathEvent(), 0.2f, 1);
    }
```
# 3. ImportProjtectAction导入工程
```csharp
public Action ImportProjtectAction;
```
```csharp
ImportProjtectAction += ImportProjectEvent;
```
```csharp
    public void ImportProject(string path)
    {
        if (string.IsNullOrEmpty(path) || !File.Exists(path)) return;
        Globals.TimerManager.Regist((v) => OpenFileCallback(new string[] { path }), 0.2f, 1);
    }
    private void ImportProjectEvent()
    {
        if (mFlag) return;
        string _title = mLocalization.GetString("ID_RTEditor_ImportProject");
        Globals.TimerManager.Regist((v) => StandaloneFileBrowser.OpenFilePanelAsync(_title, null, AppConst.ProjectExtension, false, OpenFileCallback), 0.2f, 1);        
    }
```