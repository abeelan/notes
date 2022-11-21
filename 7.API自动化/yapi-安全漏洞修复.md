Yapi 是比较好用的接口管理平台，之前写过一篇关于搭建过程的文章。



[YApi - 通过docker搭建接口管理平台](http://mp.weixin.qq.com/s?__biz=MzU2NzM4MTUxNw==&mid=2247484452&idx=1&sn=45b79a2f64343be240e20e65c772740b&chksm=fc9f5f46cbe8d650513ae789eb703b7f304f640c55b173bb959d1ba4d3ea39b09715946f4cd3&scene=21#wechat_redirect)



本篇文章，记录下使用过程中遇到的问题。



### YAPI 安全漏洞解决方法

Yapi 使用的脚本执行沙箱存在安全漏洞问题，需要更换为更安全的沙箱 safeify。详情参考 [yapi 安全漏洞详解](*https://blog.csdn.net/meifannao789456/article/details/119551053*)



也可以一并解决，断言脚本报错问题：`Error: assert.equal is not a function`。

```js
// sandbox.js 脚本需要更新如下
const Safeify = require('safeify').default;
 
module.exports = async function sandboxFn(context, script) {
    // 创建 safeify 实例
    const safeVm = new Safeify({
        timeout: 3000,
        asyncTimeout: 60000,
        // quantity: 4,          //沙箱进程数量，默认同 CPU 核数
        // memoryQuota: 500,     //沙箱最大能使用的内存（单位 m），默认 500m
        // cpuQuota: 0.5,
        // true 表示不受 CPU 限制，解决 Docker 启动问题
        unrestricted: true,
        unsafe: {
            modules: {
              // 引入assert断言库
                assert: 'assert'
            }
        }
    });
 
    safeVm.preset('const assert = require("assert");');
    
    script += "; return this;";
    // 执行动态代码
    const result = await safeVm.run(script, context);
 
    // 释放资源
    safeVm.destroy();
    return result
};
```



操作步骤

```shell
# rz 命令将 sandbox.js 传入服务器
$ rz -be
# 服务器上执行 cp 命令，覆盖原文件
$ docker cp sandbox.js yapi:/yapi/vendors/server/utils/sandbox.js
$ docker restart yapi 
```



### 单个接口调试的请求参数怎么看？

根据提示文档安装 cross-request，开发者栏 network 处查看跨域请求。



### console.log() 方法输出的日志去哪里看？

在脚本运行界面，F12 调出开发者选项，点击 console 栏查看。



### 如何抽象出公共参数？

将公共参数写到 Pre-request Script(请求参数处理脚本) 内，这样也可以实现从接口响应内获取到内容作为其他请求的参数。

```js
// Pre-request Script
// 从响应内去动态获取 session_id
let session_id =  localStorage.getItem('session_id');
context.query['session_id'] = session_id; 
console.log("当前 session_id 为：" + session_id);

// 这里可以添加公共参数
context.query['os'] = "android";


// Pre-response Script
if(context.responseData['session_id'] !== undefined) {
    let session_id = context.responseData['session_id'];
    console.log("获取到session_id:"+session_id);
    localStorage.setItem('session_id', session_id);
} /*else {
    console.log("当前响应内没有找到 session_id.");
}
*/
```



### 页面UI调整

body 示例参数框内容稍长就会折行展示，备注框太长，导致页面整体拉长，不美观。

打开开发者选项调试页面，定位到表格处，发现该处引用了 `word-break` 属性，取消勾选后查看效果，比较满意。



> [word-break](https://www.w3school.com.cn/cssref/pr_word-break.asp)：在恰当的断字点进行换行



<img src="https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210831101848946.png" alt="image-20210831101848946" style="zoom: 67%;" />



找到问题就好办了，登录容器内，找到该文件，注释掉这个属性即可。

```shell
$ docker exec -it yapi sh
$ find / -name *.css | grep index@dc0427a58a38aab0505d.css
$ vi /yapi/vendors/static/prd/index@dc0427a58a38aab0505d.css
# vi 模式内通过 /.ant-table-thead>tr>th{padding:16px 10px; 搜索关键字
# 注释掉 ant-table-thead 的自动换行属性
.ant-table-thead>tr>th{padding:16px 10px;/*word-break:break-all;*/}

# 修改后刷新页面并不能直接生效
# 原因是当前目录下还存在 index@dc0427a58a38aab0505d.css.gz 压缩文件
# 修改名字作为备份
$ mv index@dc0427a58a38aab0505d.css index@dc0427a58a38aab0505d.css_bak
# 刷新页面，修改生效
# 重新压缩文件 -v 显示命令执行过程 -k 保留原文件
$ gzip -vk index@dc0427a58a38aab0505d.css
```

![image-20210831112349198](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210831112349198.png)



### References

`[1]` yapi 安全漏洞详解: *https://blog.csdn.net/meifannao789456/article/details/119551053*
`[2]` word-break: *https://www.w3school.com.cn/cssref/pr_word-break.asp*

