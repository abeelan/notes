这篇文章记录如何给 Typora 配置 gitee 免费图床。



编写文本时，经常会插入图片，Typora 默认会把图片存储在本地，如果发布到 CSDN 或者其他在线渠道，图片就会加载失败，非常的困扰，接下来就来解决这个问题吧！

### 1. gitee

国内的代码管理远程仓库，考虑到速度问题，选它，重要的是免费；

如果没有，注册一个。

创建仓库，选择公开，初始化 README 文件。

### 2. PicGo

下载安装：https://github.com/Molunerfinn/PicGo/releases

选择跟系统对应的安装包，我这里选择的是 dmg 2.3.0 beta7 版本，傻瓜式安装。

##### 1⃣️ 安装 gitee-uploader 插件

打开后在上面菜单栏会出现一个图标，右键图标，打开详细窗口，安装插件

![image-20210813111633238](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210813111633238.png)

##### 2⃣️ 设置图床

![image-20210813113015566](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210813113015566.png)

生成令牌时，取消全选，除默认 user_info 外，勾选 projects 即可。

设置为默认图床。

### 3. Typora

偏好设置 - 图像

![image-20210813114100721](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210813114100721.png)

配置好后，验证图片是否上传成功；如果未成功，查看日志会有错误信息。
