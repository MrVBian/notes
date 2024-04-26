[TOC]

在Unity创建一个简单的http服务端，控制监听HTTP请求。

# 1. HttpListener
```csharp
            mListener = new HttpListener();
            mListener.AuthenticationSchemes = AuthenticationSchemes.Anonymous;
            mListener.Prefixes.Add(AppConst.HTTPServerUrl);
            mListener.Start();
            mListener.BeginGetContext(ListenerHandle, mListener);
```
# 2. GetTryGetResponse
```csharp
        /// <summary>
        /// 处理接受到的消息
        /// </summary>
        /// <param name="resdata"></param>
        /// <returns></returns>
        public bool GetTryGetResponse(out HttpServerReceiveMsg resdata)
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
# 3. AddHttpResponse和HandleSend
```csharp
        /// <summary>
        /// 将处理后的数据，加入答复列表，等待回复客户端
        /// </summary>
        /// <param name="msg"></param>
        public void AddHttpResponse(HttpServerResponseMsg msg)
        {
            _sendList.Enqueue(msg);
        }
```
```csharp
        /// <summary>
        /// 消息答复
        /// </summary>
        private void HandleSend()
        {
            if (_sendList.count > 0)
            {
                HttpServerResponseMsg _message = _sendList.Dequeue();
                string json = JsonMapper.ToJson(_message);

                HttpListenerResponse _res = mDic[_message.Key];

                _res.StatusCode = (int)HttpStatusCode.OK;
                _res.ContentType = "application/json;charset=UTF-8";
                _res.ContentEncoding = Encoding.UTF8;
                _res.AppendHeader("Content-Type", "text/plain;charset=UTF-8");

                // 拟返回的数据：Json格式
                using (StreamWriter writer = new StreamWriter(_res.OutputStream, Encoding.UTF8))
                {
                    writer.Write(json);
                    writer.Close();
                    _res.Close();
                }
            }
        }
```