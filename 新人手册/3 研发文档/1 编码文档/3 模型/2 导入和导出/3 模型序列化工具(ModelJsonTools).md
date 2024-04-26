[TOC]


用于自定义模型数据的创建和保存，软件模型的创建。

# 1. 设置自定义模型数据（SetModelData）
```csharp
    /// <summary>
    /// 设置自定义模型数据
    /// </summary>
    /// <param name="obj"></param>
    /// <param name="trans"></param>
    /// <param name="flag"></param>
    /// <returns></returns>
    public static ModelGameOjectData SetModelData(GameObject obj, Transform trans, bool flag)
    {
        ModelGameOjectData _data = new ModelGameOjectData();
        _data.tag = obj.tag;
        _data.layer = obj.layer;
        _data.active = obj.activeSelf;
        _data.name = obj.name;
        Component[] _com = obj.GetComponents<Component>();
        if (_com.Length != 0)
        {
            _data.componentsData = new List<ComponentDataBase>();

            for (int i = 0; i < _com.Length; i++)
            {
                if (_com[i] == null) continue;
                ComponentDataBase _componentdata = null;

                if (_com[i] is Transform)
                {
                    _componentdata = new TransformData();
                    TransformData _transdata = _componentdata as TransformData;

                    if (flag)
                    {
                        _transdata.position = Vector3.zero;
                        _transdata.angle = Vector3.zero;
                        _transdata.scale = Vector3.one;
                    }
                    else
                    {
                        _transdata.position = trans.localPosition;
                        _transdata.angle = trans.localEulerAngles;
                        _transdata.scale = trans.localScale;
                    }
                }
                else if (_com[i] is SpriteRenderer)
                {
                    SpriteRenderer _sp = _com[i] as SpriteRenderer;
                    Sprite _sprite = _sp.sprite;

                    _componentdata = new SpriteRenderData();
                    SpriteRenderData _srd = _componentdata as SpriteRenderData;
                    _srd.imageData = _sprite.texture.EncodeToPNG();
                }
                else if (_com[i] is MeshFilter)
                {
                    MeshFilter _meshfilter = _com[i] as MeshFilter;
                    if (_meshfilter.sharedMesh != null)
                    {
                        _componentdata = new MeshFilterData();
                        MeshFilterData _meshfilterdata = _componentdata as MeshFilterData;

                        Mesh _mesh = _meshfilter.sharedMesh;
                        _meshfilterdata.indexFormat = _mesh.indexFormat;
                        _meshfilterdata.MeshName = _mesh.name;
                        _meshfilterdata.vertices = _mesh.vertices;
                        _meshfilterdata.triangles = _mesh.triangles;
                        _meshfilterdata.uv = _mesh.uv;
                        //_meshfilterdata.normals = _mesh.normals;
                        //_meshfilterdata.tangents = _mesh.tangents;
                        _meshfilterdata.SubMeshCount = _mesh.subMeshCount;



                        if (_mesh.subMeshCount > 1)
                        {
                            _meshfilterdata.subMeshDescriptorData = new List<SubMeshDescriptorData>();
                            for (int j = 0; j < _mesh.subMeshCount; j++)
                            {
                                SubMeshDescriptorData _submeshdata = new SubMeshDescriptorData();

                                SubMeshDescriptor _sub = _mesh.GetSubMesh(j);
                                _submeshdata.baseVertex = _sub.baseVertex;
                                _submeshdata.firstVertex = _sub.firstVertex;
                                _submeshdata.topology = _sub.topology;

                                _submeshdata.indexStart = _sub.indexStart;

                                _submeshdata.indexCount = _sub.indexCount;

                                _submeshdata.vertexCount = _sub.vertexCount;

                                _meshfilterdata.subMeshDescriptorData.Add(_submeshdata);
                            }
                        }
                    }
                }
                else if (_com[i] is Renderer)
                {
                    Renderer _renderer = _com[i] as Renderer;
                    //Material[] _mat = _renderer.sharedMaterials;
                    Material[] _mat = null;
                    ChangeMaterials _change = obj.GetComponent<ChangeMaterials>();

                    if (_change != null)
                    {
                        _mat = _change.RecordShareMaterials;
                    }
                    else
                    {
                        _mat = _renderer.sharedMaterials;
                    }
                    if (_mat != null && _mat.Length != 0)
                    {
                        _componentdata = new RendererData();
                        RendererData _rendererdata = _componentdata as RendererData;
                        _rendererdata.disabled = !_renderer.enabled;
                        _rendererdata.materials = new List<MaterialData>();

                        for (int j = 0; j < _mat.Length; j++)
                        {
                            if (_mat[j] == null) continue;

                            MaterialData _materialdata = new MaterialData();
                            _materialdata.name = _mat[j].name;
                            _materialdata.ShaderName = _mat[j].shader.name;

                            switch (_materialdata.ShaderName)
                            {
                                case "Shader Graphs/PhysicalMaterial3DsMax":
                                    _materialdata.shaderData = new PhysicalMaterial3DsMaxShader();
                                    PhysicalMaterial3DsMaxShader _shader = _materialdata.shaderData as PhysicalMaterial3DsMaxShader;
                                    if (_change != null && _change.colors != null && j < _change.colors.Count)
                                    {
                                        _shader.baseColor = _change.colors[j];
                                    }
                                    else
                                    {
                                        _shader.baseColor = _mat[j].GetColor(PhysicalMaterial3DsMaxShader.BaseColorKey);
                                    }

                                    _shader.ReflectionsColor = _mat[j].GetColor(_shader.ReflectionsColorKey);
                                    break;
                                case "VolumetricLights/VolumetricLightURP":
                                    _materialdata.shaderData = new VolumetricLightURP();
                                    VolumetricLightURP _vlu = _materialdata.shaderData as VolumetricLightURP;
                                    if (_change != null && _change.colors != null && j < _change.colors.Count)
                                    {
                                        _vlu.baseColor = _change.colors[j];
                                    }
                                    else
                                    {
                                        _vlu.baseColor = _mat[j].GetColor(VolumetricLightURP.BaseColorKey);
                                    }

                                    break;
                                case "Universal Render Pipeline/Unlit":
                                    _materialdata.shaderData = new UnlitShader();
                                    UnlitShader _unlit = _materialdata.shaderData as UnlitShader;
                                    if (_change != null && _change.colors != null && j < _change.colors.Count)
                                    {
                                        _unlit.baseColor = _change.colors[j];
                                    }
                                    else
                                    {
                                        _unlit.baseColor = _mat[j].GetColor(UnlitShader.BaseColorKey);
                                    }
                                    break;
                                case "Universal Render Pipeline/Lit":
                                    _materialdata.shaderData = new LitShaderBase();
                                    LitShaderBase _lit = _materialdata.shaderData as LitShaderBase;
                                    if (_change != null && _change.colors != null && j < _change.colors.Count)
                                    {
                                        _lit.baseColor = _change.colors[j];
                                    }
                                    else
                                    {
                                        _lit.baseColor = _mat[j].GetColor(LitShaderBase.BaseColorKey);
                                    }
                                    _lit.SmoothnessValue = _mat[j].GetFloat(LitShaderBase.Smoothness);
                                    _lit.MetallicValue = _mat[j].GetFloat(LitShaderBase.Metallic);

                                    float _surface = _mat[j].GetFloat("_Surface");
                                    if (_surface == 1f)
                                    {
                                        _lit.transparent = true;
                                    }
                                    else
                                    {
                                        _lit.transparent = false;
                                    }
                                    break;
                                default:
                                    _materialdata.shaderData = new ShaderDataBase();
                                    break;
                            }
                            _rendererdata.materials.Add(_materialdata);
                        }
                    }
                }
                else if (_com[i] is MeshCollider)
                {
                    MeshCollider _collider = _com[i] as MeshCollider;

                    _componentdata = new MeshColliderData();

                    MeshColliderData _meshdata = _componentdata as MeshColliderData;
                    _meshdata.Convex = _collider.convex;
                    _meshdata.isTrigger = _collider.isTrigger;
                    _meshdata.enabled = _collider.enabled;
                }
                else
                {
                    if (/*_com[i] is IKController || _com[i] is IKTrussController ||*/ _com[i] is ExposeToEditor || _com[i] is ChangeMaterials)
                        continue;

                    ITypeMap _map = IOC.Resolve<ITypeMap>();
                    Type _type = _map.ToPersistentType(_com[i].GetType());
                    if (_type == null)
                        continue;
                    PersistentSurrogate<long> persistentObject = (PersistentSurrogate<long>)Activator.CreateInstance(_type);
                    persistentObject.ReadFrom(_com[i]);

                    _componentdata = new PersistentComponentData();
                    PersistentComponentData _persistentdata = _componentdata as PersistentComponentData;
                    _persistentdata.mono = persistentObject;
                }
                if (_componentdata != null)
                {
                    _componentdata.name = _com[i].GetType().AssemblyQualifiedName;
                    //Debug.Log("name--->" + _componentdata.name);
                    _data.componentsData.Add(_componentdata);
                }
            }
        }
        if (string.IsNullOrEmpty(_data.name))
        {
            _data.name = obj.name;
        }
        if (trans.childCount != 0)
        {
            _data.childGameObjects = new List<ModelGameOjectData>();
            for (int i = 0; i < trans.childCount; i++)
            {
                Transform _trans = trans.GetChild(i);
                GameObjectSaveTag _savetag = _trans.GetComponent<GameObjectSaveTag>();
                if (_savetag != null)
                    continue;

                GameObject _obj = _trans.gameObject;
                _data.childGameObjects.Add(SetModelData(_obj, _trans, false));
            }
        }
        return _data;
    }
```
# 2. 根据自定义模型数据创建模型和模型子节点及添加脚本
```csharp
    /// <summary>
    /// 根据数据，创建模型
    /// </summary>
    /// <param name="gameobjectdata"></param>
    /// <returns></returns>
    public static GameObject CreateGameObject(ModelGameOjectData gameobjectdata)
    {
        GameObject _obj = new GameObject(gameobjectdata.name);
        _obj.tag = gameobjectdata.tag;
        _obj.layer = gameobjectdata.layer;
        _obj.SetActive(gameobjectdata.active);
        Transform _trans = _obj.transform;


        if (gameobjectdata.childGameObjects != null && gameobjectdata.childGameObjects.Count != 0)
        {
            Dictionary<GameObject, List<ComponentDataBase>> _dic = new Dictionary<GameObject, List<ComponentDataBase>>();
            for (int i = 0; i < gameobjectdata.childGameObjects.Count; i++)
            {
                CreateChildGameObject(gameobjectdata.childGameObjects[i], _trans, _dic);
            }

            List<GameObject> _list = new List<GameObject>(_dic.Keys);

            for (int i = _list.Count - 1; i >= 0; i--)
            {
                for (int j = 0; j < _dic[_list[i]].Count; j++)
                {
                    AddComponets(_list[i], _list[i].transform, _dic[_list[i]][j]);
                }

            }
        }

        if (gameobjectdata.componentsData != null && gameobjectdata.componentsData.Count != 0)
        {
            bool _flag = false;
            for (int i = 0; i < gameobjectdata.componentsData.Count; i++)
            {
                if (!_flag)
                {
                    if (typeof(RobotModel) == Type.GetType(gameobjectdata.componentsData[i].name))
                        _flag = true;
                }

                AddComponets(_obj, _trans, gameobjectdata.componentsData[i]);
            }

            if (_flag)
            {
                List<Transform> _list = new List<Transform>();
                UtilsLogic.GetTransformByTag(_trans, ToolsAssembleState.AssemblePosTag, ref _list, false);
                _list.ForEach((v) =>
                {
                    var _exe = v.GetComponent<ExposeToEditor>();
                    if (_exe != null)
                    {
                        _exe.CanTransform = true;
                        _exe.CanInspect = true;
                    }
                });
            }
        }

        ExposeToEditor _exp = _obj.AddComponent<ExposeToEditor>();
        SetModelExposeToEditor(_exp, true);

        GameObjectSaveTag _tag = _obj.GetComponent<GameObjectSaveTag>();
        if (_tag == null)
        {
            _tag = _obj.AddComponent(typeof(GameObjectSaveTag)) as GameObjectSaveTag;
            _tag.flag = true;
        }
        return _obj;
    }
    /// <summary>
    /// 添加脚本
    /// </summary>
    /// <param name="_obj"></param>
    /// <param name="_trans"></param>
    /// <param name="componentData"></param>
    private static void AddComponets(GameObject _obj, Transform _trans, ComponentDataBase componentData)
    {

        //Debug.Log("name---->" + gameobjectdata.componentsData[i].name);
        string _name = componentData.name;
        var _script = Type.GetType(_name);
        if (_script == null)
        {
            Debug.LogWarning(_name + " :: This Componet is Missing");
            return;
        }

        if (_script == typeof(Transform))
        {
            TransformData _transdata = componentData as TransformData;
            _trans.localPosition = _transdata.position;
            _trans.localEulerAngles = _transdata.angle;
            _trans.localScale = _transdata.scale;
        }
        else if (_script == typeof(SpriteRenderer))
        {
            var _com = _obj.AddComponent(_script);
            SpriteRenderer _sp = _com as SpriteRenderer;
            SpriteRenderData _srd = componentData as SpriteRenderData;
            Texture2D _tex = new Texture2D(10, 10);
            _tex.LoadImage(_srd.imageData);
            _sp.sprite = Sprite.Create(_tex, new Rect(0, 0, _tex.width, _tex.height), Vector2.zero);
        }
        else if (_script == typeof(MeshFilter))
        {
            var _com = _obj.AddComponent(_script);

            MeshFilter _meshfilter = _com as MeshFilter;
            MeshFilterData _meshfilterdata = componentData as MeshFilterData;

            Mesh _mesh = new Mesh();
            _mesh.indexFormat = _meshfilterdata.indexFormat;
            _mesh.name = _meshfilterdata.MeshName;
            _mesh.vertices = _meshfilterdata.vertices;
            _mesh.triangles = _meshfilterdata.triangles;
            //_mesh.normals = _meshfilterdata.normals;
            //_mesh.tangents = _meshfilterdata.tangents;
            _mesh.uv = _meshfilterdata.uv;
            _mesh.subMeshCount = _meshfilterdata.SubMeshCount;
            if (_meshfilterdata.SubMeshCount > 1)
            {
                for (int j = 0; j < _meshfilterdata.SubMeshCount; j++)
                {
                    SubMeshDescriptor _sub = new SubMeshDescriptor();
                    SubMeshDescriptorData _submeshdata = _meshfilterdata.subMeshDescriptorData[j];
                    _sub.baseVertex = _submeshdata.baseVertex;
                    _sub.firstVertex = _submeshdata.firstVertex;
                    _sub.topology = _submeshdata.topology;
                    _sub.indexStart = _submeshdata.indexStart;
                    _sub.indexCount = _submeshdata.indexCount;
                    _sub.vertexCount = _submeshdata.vertexCount;
                    _mesh.SetSubMesh(j, _sub);
                }
            }
            _mesh.RecalculateTangents();
            //从顶点重新计算网格的边界体积。
            _mesh.RecalculateNormals();
            //从三角形和顶点重新计算网格的法线
            _mesh.RecalculateBounds();
            _meshfilter.mesh = _mesh;
        }
        else if (_script == typeof(MeshRenderer))
        {
            RendererData _rendererdata = componentData as RendererData;

            var _com = _obj.AddComponent(_script);

            Material[] materials = new Material[_rendererdata.materials.Count];

            for (int j = 0; j < _rendererdata.materials.Count; j++)
            {
                Shader _shader = Shader.Find(_rendererdata.materials[j].ShaderName);
                materials[j] = new Material(_shader);
                materials[j].name = _rendererdata.materials[j].name;

                switch (_rendererdata.materials[j].ShaderName)
                {
                    case "Shader Graphs/PhysicalMaterial3DsMax":
                        PhysicalMaterial3DsMaxShader _3DsMax = _rendererdata.materials[j].shaderData as PhysicalMaterial3DsMaxShader;
                        materials[j].SetColor(PhysicalMaterial3DsMaxShader.BaseColorKey, _3DsMax.baseColor);
                        materials[j].SetColor(_3DsMax.ReflectionsColorKey, _3DsMax.ReflectionsColor);
                        break;
                    case "VolumetricLights/VolumetricLightURP":
                        VolumetricLightURP _vlu = _rendererdata.materials[j].shaderData as VolumetricLightURP;
                        break;
                    case "Universal Render Pipeline/Unlit":
                        UnlitShader _unlit = _rendererdata.materials[j].shaderData as UnlitShader;
                        materials[j].SetColor(UnlitShader.BaseColorKey, _unlit.baseColor);
                        break;
                    case "Universal Render Pipeline/Lit":
                        LitShaderBase _lit = _rendererdata.materials[j].shaderData as LitShaderBase;
                        materials[j].SetColor(LitShaderBase.BaseColorKey, _lit.baseColor);
                        materials[j].SetFloat(LitShaderBase.Smoothness, _lit.SmoothnessValue);
                        materials[j].SetFloat(LitShaderBase.Metallic, _lit.MetallicValue);
                        UtilsLogic.ChangeTransparency(materials[j], _lit.transparent);
                        break;
                    default:
                        ShaderDataBase _shaderData = _rendererdata.materials[j].shaderData;
                        break;
                }
                //materials[j].SetColor("_BaseColor", _rendererdata.materials[j].shaderData.baseColor);
            }

            MeshRenderer _renderer = _com as MeshRenderer;
            _renderer.materials = materials;
            _renderer.enabled = !_rendererdata.disabled;
        }
        else if (_script == typeof(MeshCollider))
        {
            if (_obj.GetComponent<MeshCollider>() != null)
                return;
            MeshColliderData _coldata = componentData as MeshColliderData;
            var _com = _obj.AddComponent<MeshCollider>();
            MeshCollider _collider = _com as MeshCollider;
            _collider.convex = _coldata.Convex;
            _collider.isTrigger = _coldata.isTrigger;
            _collider.enabled = _coldata.enabled;
        }
        else
        {
            PersistentComponentData _persistentdata = componentData as PersistentComponentData;
            var _com = _obj.AddComponent(_script);
            _persistentdata.mono.WriteTo(_com);
        }
    }

    /// <summary>
    /// 创建模型节点
    /// </summary>
    /// <param name="gameobjectdata"></param>
    /// <param name="parent"></param>
    /// <param name="dic"></param>
    private static void CreateChildGameObject(ModelGameOjectData gameobjectdata, Transform parent, Dictionary<GameObject, List<ComponentDataBase>> dic)
    {
        GameObject _obj = new GameObject(gameobjectdata.name);
        _obj.tag = gameobjectdata.tag;
        _obj.layer = gameobjectdata.layer;
        _obj.SetActive(gameobjectdata.active);

        Transform _trans = _obj.transform;
        _trans.SetParent(parent);
        if (gameobjectdata.componentsData != null && gameobjectdata.componentsData.Count != 0)
        {
            dic.Add(_obj, gameobjectdata.componentsData);
        }

        if (gameobjectdata.childGameObjects != null && gameobjectdata.childGameObjects.Count != 0)
        {
            for (int i = 0; i < gameobjectdata.childGameObjects.Count; i++)
            {
                CreateChildGameObject(gameobjectdata.childGameObjects[i], _trans, dic);
            }
        }

        ExposeToEditor _exp = _obj.AddComponent<ExposeToEditor>();
        SetModelExposeToEditor(_exp, false);

        if (_obj.CompareTag(ToolsAssembleState.AssemblePosTag))
        {
            _exp.CanBeParent = false;
        }
        else
        {
            _exp.CanBeParent = true;
        }
    }
```

# 3.GetModelDataFromJson
```csharp
    /// <summary>
    /// 开启线程，根据文件路径，反序列化自定义模型数据
    /// </summary>
    /// <param name="path"></param>
    /// <param name="action"></param>

    public static void GetModelDataFromJson(string path, Action<ModelGameOjectData> action)
    {
        mAction = action;
        Task.Run(() =>
        {
            ModelGameOjectData _data = SerializeTool.GetBinaryDeserializeFromFile<ModelGameOjectData>(path);

            mAction?.Invoke(_data);
        });
    }
```
# 4.litjson获得反序列化数据 (GetJsonDeserialize)
```csharp
    /// <summary>
    /// litjson获得反序列化数据 
    /// </summary>
    /// <typeparam name="T"></typeparam>
    /// <param name="path"></param>
    /// <returns></returns>
    public static T GetJsonDeserialize<T>(string path)
    {
        T _t;
        using (FileStream s = File.Open(path, FileMode.Open))
        {
            using (StreamReader sr = new StreamReader(s))
            {
                string _content = sr.ReadToEnd();
                _t = LitJson.JsonMapper.ToObject<T>(_content);
                sr.Close();
            }
            s.Close();
        }
        return _t;
    }
```