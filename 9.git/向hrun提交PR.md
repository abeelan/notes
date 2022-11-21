> 记录向开源项目提交 PR.



### 准备

- 创建 github 账号
- 网络通畅



### 开始

1、进入项目主页，以`Httprunner`为例。

```
https://github.com/httprunner/httprunner
```

2、点击项目右上角的 `Fork` 按钮，把项目克隆到自己的远程仓库，方便后续修改提交。

点击后页面会自动跳转到自己的远程仓库，把代码 `Clone` 到本地。

```bash
# 查看远程仓库链接
$ git remote -v                                               
origin  git@github.com:abeelan/httprunner.git (fetch)
origin  git@github.com:abeelan/httprunner.git (push)
```

3、与原项目建立链接

```bash
$ git remote add upstream git@github.com:httprunner/httprunner.git

$ git remote -v                                                   
origin  git@github.com:abeelan/httprunner.git (fetch)
origin  git@github.com:abeelan/httprunner.git (push)
upstream        git@github.com:httprunner/httprunner.git (fetch)
upstream        git@github.com:httprunner/httprunner.git (push)
```

4、创建新的分支，修复问题

```bash
$ git checkout -b bug-abee  
Switched to a new branch 'bug-abee'
```

- .env 文件未忽略注释及空行；
- HTTPS 请求无法获取客户端和服务端的 IP 、端口号。

5、提交修复后的分支代码到远程仓库

```bash
$ git add .
$ git commit -m "bug fixed"

# 提交到远程仓库
$ git push origin bug-abee
To github.com:abeelan/httprunner.git
 * [new branch]      bug-abee -> bug-abee
```

6、进入远程仓库页面，发起 PR 请求。

![](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20220118174247050.png)

点击按钮，自动跳转到原项目仓库创建 PR 页面。

![](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20220118175652398.png)

编辑完成后，点击创建即可。

刷新页面，在 PR 列表就能看到了，静待项目管理员处理吧～

