接口鉴权的通用解决方案：

- 认证信息的获取
- 认证信息的携带



<img src="/Users/lan/Library/Application Support/typora-user-images/image-20221023224410166.png" alt="image-20221023224410166" style="zoom:50%;" />



auth 鉴权请求方法

```python
import requests
from requests.auth import HTTOBasicAuth

proxy = {
    "http": "http://127.0.0.1:8888",
    "https": "https://127.0.0.1:8888"
}

r = requests.get(
    url="https://baidu.com", 
    proxies=proxy, 
    verify=False, 
    auth=HttpBasicAuth("username", "password")
)
```

