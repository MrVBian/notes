[TOC]


HttpClient用于HTTP网络之间通信。

# 1. Post方法
```csharp
        private void Post(HttpWebRequestMsg reqdata, ref HttpWebResponseMsg resdata)
        {
            HttpWebRequest req;
            HttpWebResponse res;
            //encode = encode ?? Encoding.UTF8;
            req = (HttpWebRequest)WebRequest.Create(new Uri(reqdata.url));
            req.ContentType = "application/json";
            req.Method = "POST";
            req.Timeout = TimeOut;

            try
            {
                if (reqdata.data != null)
                {
                    string _postdata = LitJson.JsonMapper.ToJson(reqdata.data);
                    if (AppConst.showLog)
                    {
                        Debug.Log("postjson------->" + _postdata);
                    }                    
                    byte[] byteArray = encode.GetBytes(_postdata);
                    Stream stream = req.GetRequestStream();
                    stream.Write(byteArray, 0, byteArray.Length);
                    stream.Close();
                }

                res = (HttpWebResponse)req.GetResponse();
                resdata.code = (int)res.StatusCode;
                using (Stream resStream = res.GetResponseStream())
                {
                    string _response = GetStrResponse(resStream, encry, encode);
                    if (!string.IsNullOrEmpty(_response))
                    {
                        if (AppConst.showLog)
                        {
                            Debug.Log("response----->" + _response);
                        }                        
                        Type serializerType;
                        if (reqdata.data.SerializerType == null)
                        {
                            serializerType = typeof(HttpWebRequestDataBase);
                        }
                        else
                        {
                            serializerType = reqdata.data.SerializerType;
                        }
                        // Debug.Log("serializerType: " + reqdata.data.SerializerType.ToString() + "  " + serializerType.ToString());
                        object _data = LitJson.JsonMapper.ToObject(_response, serializerType);
                        if (_data != null && _data is HttpWebResponseDataBase)
                        {
                            resdata.data = _data as HttpWebResponseDataBase;
                        }
                    }
                }
                res.Close();
            }
            catch (WebException ex)
            {
                resdata.code = (int)ex.Status;
            }

            req.Abort();
        }
```
# 2. Get方法
```csharp
        private void Get(HttpWebRequestMsg reqdata, ref HttpWebResponseMsg resdata)
        {
            HttpWebRequest req = null;
            HttpWebResponse res = null;
            req = (HttpWebRequest)WebRequest.Create(reqdata.url);
            req.AllowAutoRedirect = false;
            req.Timeout = TimeOut;

            try
            {
                res = (HttpWebResponse)req.GetResponse();
            }
            catch (WebException ex)
            {
                res = (HttpWebResponse)ex.Response;
            }

            resdata.code = ((int)res.StatusCode);
            using (Stream stream = res.GetResponseStream())
            {
                string _response = GetStrResponse(stream, encry, encode);
                if (!string.IsNullOrEmpty(_response))
                {
                    object _data = LitJson.JsonMapper.ToObject(_response, reqdata.data.SerializerType);
                    if (_data is HttpWebResponseDataBase)
                    {
                        resdata.data = _data as HttpWebResponseDataBase;
                    }
                }
            }

            res.Close();
            req.Abort();
        }
```

# 3. AddHttpReq
```csharp
        /// <summary>
        /// 加入网络请求列表
        /// </summary>
        /// <param name="data"></param>
        public void AddHttpReq(HttpWebRequestMsg data)
        {
            _sendList.Enqueue(data);
        }
```

# 4. TryGetResponse
```csharp
        /// <summary>
        /// 尝试获得网络响应数据
        /// </summary>
        /// <param name="resdata"></param>
        /// <returns></returns>
        public bool TryGetResponse(out HttpWebResponseMsg resdata)
        {
            if (_receiveList.count > 0)
            {
                resdata = _receiveList.Dequeue();
                return true;
            }
            else
            {
                resdata = null;
                return false;
            }
        }
```