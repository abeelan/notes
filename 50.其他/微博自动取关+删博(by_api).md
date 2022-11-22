> 通过 API 删除微博、取消关注用户。

五一假期在家呆着无聊，收拾卫生，屋子里弄得是干干净净，一尘不染，躺在床上，打开微博刷起热搜。

这是我的微博账号。

![6651651806823_.pic](https://gitee.com/abeelan/image-hosting-service/raw/master/img/6651651806823_.pic.jpg)

等等... 点了这么多关注，发了这么多微博！？

原来我的账号一直被媳妇当作测试账号来用，几年下来增加了很多测试微博和无效关注，手动删除太痛苦了，利用一个下午的时间编写自动化脚本，清扫账号。



**技术栈**

- Requests：发送 API 请求
- Requests-html：解析网页，提取值
- CSS 定位、XPath 定位或正则，都可
- Chrome Network 抓包



都是一些基础用法，没啥难度，算是一个小练习吧。





### 准备工作

第一步是获取登录凭证。

打开谷歌浏览器，进入微博网页移动版(`weibo.cn`)，登录自己的账号。按 `F12` 进入开发者工具，点击 `Network` 开启抓包，刷新页面，在 API 请求头内，把 `cookie` 内容复制下来。

![image-20220506113334605](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20220506113334605.png)

第二步抓取参数 `st` 的值。

首页 -> 关注，到达关注列表页面，用箭头定位到【取消关注】元素，该元素的 `href` 属性链接内就包含 `st` 的值，记录一下。 



### 流程分析

> 以取消关注为例，删除微博同理。

开启抓包，手动取消关注一个用户，得到请求如下：

```
https://weibo.cn/attention/del?rl=1&act=delc&uid=5623715908&st=e08ff9
```

- rl：代表什么意思暂时不知道，当取消关注其他用户时，该值不变，证明可复用；
- act：按字面意思，应该是操作类型，表示删除的动作，该值不变；
- uid：要取消关注的用户的 ID，定位到该元素，发现在 `href` 元素内包含该参数，可以通过脚本实时获取；
- st：与上面我们抓到的是一样的，当取消关注其他用户时，该值不变，证明可复用。



初步分析完成后，程序的大致流程也就出来了：

- 进入网页，登录
- 进入关注列表页面，获取分页总数
- 遍历分页总数次，获取每页用户信息
- 遍历每个用户，提取到 uid
- 执行取消关注请求



### 脚本编写

首先做一些初始化工作，比如请求链接拼装、实例变量初始化、获取当前登录用户信息。

```python
class WeiBo:

    HOST = "https://weibo.cn"
    USER_AGENT = ""

    def __init__(self, uid, flag=1):
        self.headers = {
            'user-agent': self.USER_AGENT,
            'cookie': COOKIES
        }
        self.session = HTMLSession()
        self.session.headers = self.headers

        self.uid = str(uid)
        self.flag = int(flag)  # 1 删除全部微博；2 取消全部关注
        self.total_weibo_pages = 0   # 已发表微博总页数，每页 10 条
        self.total_follow_pages = 0  # 已关注用户总页数，每页 10 个

        # URL
        self.url_info = f"{self.HOST}/{self.uid}/info"
        self.url_profile = f"{self.HOST}/{self.uid}/profile"  # Default page=1
        self.url_del_weibo = f"{self.HOST}/mblog/del"
        self.url_follow = f"{self.HOST}/{self.uid}/follow"
        self.url_del_follow = f"{self.HOST}/attention/del"

        # 用户信息，从 profile 获取
        self.post = 0  # 发表微博数
        self.follow = 0  # 关注人数
        self.fans = 0  # 粉丝人数

        # 初始化
        self.get_weibo_list_page()
        self.get_user_info()
```


实现方法：需包含两个功能，获取关注总页数、根据页数获取该页所有用户信息。

```python
def get_follow_list_page(self, page=1):
    """ 根据页数获取我的关注页面的内容
        [<Element>, ...]
    """
    r = self.session.get(self.url_follow, params={"page": page})
    html = r.html
    # 获取总页数
    if self.total_follow_pages == 0:
        page_nums_str: str = html.find("div.pa", first=True).text
        self.total_follow_pages = int(page_nums_str.split("/")[-1].replace("页", ""))
    # 获取被关注人对象（每页最多 10 个）
    follow_objs = html.find("table tr")
    return follow_objs
```



实现方法：根据单个用户，提取到`UID`。

```python
def get_single_uid(self, user_obj: Element):
    """获取被关注用户 ID"""
    uid = None

    elements = user_obj.find("a")
    name = elements[1].text
    for e in elements:
        link = e.attrs["href"]
        if "uid" in link:
            params = link.split("?")[-1].split("&")
            for p in params:
                if "uid" in p:
                    uid = p.split("=")[-1]
                    break

    logger.info(f"当前被关注用户为: {name}({uid})")
    return uid
```



实现方法：根据`UID`，取消关注。

```python
def del_single_follow(self, uid):
    """取消关注, uid 为被关注用户的 ID"""
    headers = {
        **self.headers,
        **{"referer": self.url_del_follow + f"?uid={uid}&rl=1&st={ST}"}
    }
    params = {
        "rl": "1",
        "act": "delc",
        "uid": uid,
        "st": ST
    }
    r = requests.get(self.url_del_follow, params=params, headers=headers)
    # logger.debug(r.url)
    # logger.debug(r.text)
    if r.status_code == 200 and "首页" not in r.text:
        logger.success(f"取消关注成功: {uid}")
        return True
    return False
```



将上面实现的方法，组装一下，通过遍历完成取消全部关注的功能。

```python
def del_all_follow(self):
    """取消全部关注"""
    self.get_follow_list_page()  # 获取关注总页数
    del_nums = 0  # 取消关注数统计

    for i in range(1, self.total_follow_pages):
        logger.info(f"******* 取消关注，第 {i} 页 *******")
        objs = self.get_follow_list_page(page=i)

        for obj in objs:
            uid = self.get_single_uid(obj)
            if self.del_single_follow(uid):
                del_nums += 1
                logger.info(f"取消关注总人数为: {del_nums}.\n")
                sleep(sec)
```



运行程序，会输出基本信息。

![image-20220506121358689](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20220506121358689.png)

此时，访问网页版，关注人数会实时减少，证明程序功能正常。

实际运行过程中， 当请求`100+`次后，会被限制访问一段时间。

最无语的是，还剩最后 20 个关注，触发了风控策略，关注列表不返回内容。不过还好，总体上达到预期，给我省了不少功夫。

![6661651817568_.pic](https://gitee.com/abeelan/image-hosting-service/raw/master/img/6661651817568_.pic.jpg)

使用微博客户端过程中，还发现了一些体验不好的情况，比如微博手机客户端我的页面，不支持下拉刷新；已经取关的用户，依然在微博-关注里面展示动态；还有大半屏的消息通知开关...好复杂。

只是一个普通用户，了解的不多，吐槽太片面，哈哈。

最后，源码地址：
https://github.com/abeelan/ha-ha