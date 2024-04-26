[TOC]

# 1. GetInspectorRotationValueMethod
```csharp
    /// <summary>
    /// 获得Inspector面板局部角度数据
    /// </summary>
    /// <param name="transform"></param>
    /// <returns></returns>
    public static Vector3 GetInspectorRotationValueMethod(Transform transform)
    {
        // 获取原生值
        System.Type transformType = transform.GetType();
        PropertyInfo m_propertyInfo_rotationOrder = transformType.GetProperty("rotationOrder", BindingFlags.Instance | BindingFlags.NonPublic);
        object m_OldRotationOrder = m_propertyInfo_rotationOrder.GetValue(transform, null);
        MethodInfo m_methodInfo_GetLocalEulerAngles = transformType.GetMethod("GetLocalEulerAngles", BindingFlags.Instance | BindingFlags.NonPublic);
        object value = m_methodInfo_GetLocalEulerAngles.Invoke(transform, new object[] { m_OldRotationOrder });
        string temp = value.ToString();
        //将字符串第一个和最后一个去掉
        temp = temp.Remove(0, 1);
        temp = temp.Remove(temp.Length - 1, 1);
        //用‘，’号分割
        string[] tempVector3;
        tempVector3 = temp.Split(',');
        //将分割好的数据传给Vector3
        Vector3 vector3 = new Vector3(float.Parse(tempVector3[0]), float.Parse(tempVector3[1]), float.Parse(tempVector3[2]));
        return vector3;
    }
```

# 2. ChangeTransparency(设置材质球)
```csharp
    public enum SurfaceType
    {
        Opaque,
        Transparent
    }
    public enum BlendMode
    {
        Alpha,
        Premultiply,
        Additive,
        Multiply
    }
    /// <summary>
    /// 设置URP中的Lit Shader，transparent为是否透明
    /// </summary>
    /// <param name="wallMaterial"></param>
    /// <param name="transparent"></param>
    public static void ChangeTransparency(Material wallMaterial, bool transparent)
    {
        if (transparent)
        {
            wallMaterial.SetFloat("_Surface", (float)SurfaceType.Transparent);
            wallMaterial.SetFloat("_Blend", (float)BlendMode.Alpha);
        }
        else
        {
            wallMaterial.SetFloat("_Surface", (float)SurfaceType.Opaque);
        }
        SetupMaterialBlendMode(wallMaterial);
    }
    private static void SetupMaterialBlendMode(Material material)
    {
        if (material == null)
            throw new ArgumentNullException("material");
        bool alphaClip = material.GetFloat("_AlphaClip") == 1;
        if (alphaClip)
            material.EnableKeyword("_ALPHATEST_ON");
        else
            material.DisableKeyword("_ALPHATEST_ON");
        SurfaceType surfaceType = (SurfaceType)material.GetFloat("_Surface");
        if (surfaceType == 0)
        {
            material.SetOverrideTag("RenderType", "");
            material.SetInt("_SrcBlend", (int)UnityEngine.Rendering.BlendMode.One);
            material.SetInt("_DstBlend", (int)UnityEngine.Rendering.BlendMode.Zero);
            material.SetInt("_ZWrite", 1);
            material.DisableKeyword("_ALPHAPREMULTIPLY_ON");
            material.renderQueue = -1;
            material.SetShaderPassEnabled("ShadowCaster", true);
        }
        else
        {
            BlendMode blendMode = (BlendMode)material.GetFloat("_Blend");
            switch (blendMode)
            {
                case BlendMode.Alpha:
                    material.SetOverrideTag("RenderType", "Transparent");
                    material.SetInt("_SrcBlend", (int)UnityEngine.Rendering.BlendMode.SrcAlpha);
                    material.SetInt("_DstBlend", (int)UnityEngine.Rendering.BlendMode.OneMinusSrcAlpha);
                    material.SetInt("_ZWrite", 0);
                    material.DisableKeyword("_ALPHAPREMULTIPLY_ON");
                    material.renderQueue = (int)UnityEngine.Rendering.RenderQueue.Transparent;
                    material.SetShaderPassEnabled("ShadowCaster", false);
                    break;
                case BlendMode.Premultiply:
                    material.SetOverrideTag("RenderType", "Transparent");
                    material.SetInt("_SrcBlend", (int)UnityEngine.Rendering.BlendMode.One);
                    material.SetInt("_DstBlend", (int)UnityEngine.Rendering.BlendMode.OneMinusSrcAlpha);
                    material.SetInt("_ZWrite", 0);
                    material.EnableKeyword("_ALPHAPREMULTIPLY_ON");
                    material.renderQueue = (int)UnityEngine.Rendering.RenderQueue.Transparent;
                    material.SetShaderPassEnabled("ShadowCaster", false);
                    break;
                case BlendMode.Additive:
                    material.SetOverrideTag("RenderType", "Transparent");
                    material.SetInt("_SrcBlend", (int)UnityEngine.Rendering.BlendMode.One);
                    material.SetInt("_DstBlend", (int)UnityEngine.Rendering.BlendMode.One);
                    material.SetInt("_ZWrite", 0);
                    material.DisableKeyword("_ALPHAPREMULTIPLY_ON");
                    material.renderQueue = (int)UnityEngine.Rendering.RenderQueue.Transparent;
                    material.SetShaderPassEnabled("ShadowCaster", false);
                    break;
                case BlendMode.Multiply:
                    material.SetOverrideTag("RenderType", "Transparent");
                    material.SetInt("_SrcBlend", (int)UnityEngine.Rendering.BlendMode.DstColor);
                    material.SetInt("_DstBlend", (int)UnityEngine.Rendering.BlendMode.Zero);
                    material.SetInt("_ZWrite", 0);
                    material.DisableKeyword("_ALPHAPREMULTIPLY_ON");
                    material.renderQueue = (int)UnityEngine.Rendering.RenderQueue.Transparent;
                    material.SetShaderPassEnabled("ShadowCaster", false);
                    break;
            }
        }
    }
```
# 3. GetTransformByTag
```csharp
    /// <summary>
    /// 获得Tag标签Transform列表
    /// </summary>
    /// <param name="transform"></param>
    /// <param name="tag"></param>
    /// <param name="list"></param>
    /// <param name="isDeepth"></param>
    public static void GetTransformByTag(Transform transform, string tag, ref List<Transform> list, bool isDeepth = true)
    {
        if (/*!transform.gameObject.activeSelf ||*/ string.IsNullOrEmpty(tag)) return;
        if (transform.tag.Equals(tag))
        {
            list.Add(transform);
        }
        if (transform.childCount != 0)
        {
            for (int i = 0; i < transform.childCount; i++)
            {
                Transform _child = transform.GetChild(i);
                GameObjectSaveTag _tag = _child.GetComponent<GameObjectSaveTag>();
                if (_tag != null && !_tag.flag)
                    continue;

                if (isDeepth)
                {
                    GetTransformByTag(_child, tag, ref list, isDeepth);
                }
                else
                {
                    ModelGameobjectBase _base = _child.GetComponent<ModelGameobjectBase>();
                    if (_base == null)
                    {
                        GetTransformByTag(_child, tag, ref list, isDeepth);
                    }
                }
            }
        }
    }
```

# 4. GetStringByEncoding
```csharp
    /// <summary>
    /// 根据编码获得字符串
    /// </summary>
    /// <param name="content"></param>
    /// <param name="encoding"></param>
    /// <returns></returns>
    public static string GetStringByEncoding(string content, Encoding encoding)
    {
        var buffer = Encoding.UTF8.GetBytes(content);
        buffer = Encoding.Convert(Encoding.UTF8, encoding, buffer);
        return encoding.GetString(buffer);
    }
```