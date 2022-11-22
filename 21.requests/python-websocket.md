> websocket 测试



```python
from websocket import create_connection
# 测试地址：http://www.websocket-test.com/

def websocket_protocol():
    # 发起连接
    ws = create_connection("ws://IP或域名:端口")
    # 获取服务器返回信息
    result = ws.recv()
    print(result)
    # 发送请求信息
    ws.send("hello world")
    result_send = ws.recv()
    print(result_send)
    ws.close()
```

