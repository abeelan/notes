博客静态页面搭建完成后，接下来就需要对外发布展示。但是还没有自己的服务器和注册域名，如何发布呢？

这里要感谢 github 提供的 `github.io`，可以做到这一点，并且免费。



## 1. 创建 github.io

首先 [注册](https://github.com/) 创建账号；登录 `github` 个人页面，新建仓库

这里新建两个仓库：

- **blog**

  这个仓库用于存在 Hugo 博客的内容文件，如果本地存储的话，不创建也是可以的

- **用户名.github.io**

  对外发布的仓库，这是 github 免费提供的可访问域名，仓库名格式必须为 `用户名.github.io`，用户名可以通过点击头像查看

  创建完成后，进入该仓库的设置页面内，设置一下页面模版，不然会展示不出来网页样式

  ![image-20210121141839306](/Users/lan/Library/Application Support/typora-user-images/image-20210121141839306.png)



## 2. 关联远程仓库

- **blog 关联远程仓库**

  进入博客目录，关联远程仓库

  ```shell
  $ cd $blogPath
  $ git remote add orgin https://github.com/$blogProject
  ```

- **用户名 .github.io 关联远程仓库**

  将 hugo 内容转为静态网页文件，会在当前目录生成 `public` 目录

  ```shell
  $ hugo --theme=sam --buildDrafts --baseUrl="https://name.github.io/"
  $ cd public
  $ git remote add origin git@github.com:$name/$name.github.io.git
  $ git add -A
  $ git commit -m "first commit"
  $ git push -u origin master
  ```



## 遇到的报错

```shell
$ hugo --theme=sam --buildDrafts --baseUrl="https://name.github.io/"

ERROR 2021/01/20 22:38:04 error: failed to transform resource: POSTCSS: failed to transform "css/main.css" (text/css): PostCSS not found; install with "npm install postcss-cli". See https://gohugo.io/hugo-pipes/postcss/
```

按照提示 `npm install postcss-cli` 进行下载：

```shell
$ npm install postcss-cli
```

下载完成后，再次执行，依然报错：

```shell
$ hugo --theme=sam --buildDrafts --baseUrl="https://name.github.io/"

Building sites … /Users/lanyangbi.blog/node_modules/fs-extra/lib/mkdirs/make-dir.js:85
      } catch {
              ^
SyntaxError: Unexpected token {
    at createScript (vm.js:80:10)
    at Object.runInThisContext (vm.js:139:10)
    at Module._compile (module.js:617:28)
    at Object.Module._extensions..js (module.js:664:10)
    at Module.load (module.js:566:32)
    at tryModuleLoad (module.js:506:12)
    at Function.Module._load (module.js:498:3)
    at Module.require (module.js:597:17)
    at require (internal/module.js:11:18)
    at Object.<anonymous> (/Users/lanyangbi.blog/node_modules/fs-extra/lib/mkdirs/index.js:3:44)
ERROR 2021/01/21 14:16:51 error: failed to transform resource: exit status 1
Total in 231 ms
Error: Error building site: logged 1 error(s)
```

该问题原因是 node 版本过低，进行升级：

```shell
# 查看 node 版本号
$ node -v
v8.14.1

# 查看 node 发布版本
$ nvm list-remote 

# 安装最新版，通过淘宝镜像下载更快
$ export NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/mirrors/node
$ nvm install v15.6.0

# 切换 node 版本，第二行是将该版本设置为默认版本
$ nvm use v15.6.0
$ nvm alias default v15.6.0
```

升级 node 版本后，依然报错

```shell
$ hugo --theme=sam --buildDrafts --baseUrl="https://name.github.io/"

Building sites … node:internal/modules/cjs/loader:928
  throw err;
  ^
Error: Cannot find module 'postcss'
Require stack:
- /Users/name.blog/node_modules/postcss-cli/index.js
- /Users/lanyangbi.blog/node_modules/postcss-cli/bin/postcss
    at Function.Module._resolveFilename (node:internal/modules/cjs/loader:925:15)
    at Function.Module._load (node:internal/modules/cjs/loader:769:27)
    at Module.require (node:internal/modules/cjs/loader:997:19)
    at require (node:internal/modules/cjs/helpers:92:18)
    at Object.<anonymous> (/Users/lanyangbi.blog/node_modules/postcss-cli/index.js:14:17)
    at Module._compile (node:internal/modules/cjs/loader:1108:14)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1137:10)
    at Module.load (node:internal/modules/cjs/loader:973:32)
    at Function.Module._load (node:internal/modules/cjs/loader:813:14)
    at Module.require (node:internal/modules/cjs/loader:997:19) {
  code: 'MODULE_NOT_FOUND',
  requireStack: [
    '/Users/name.blog/node_modules/postcss-cli/index.js',
    '/Users/name.blog/node_modules/postcss-cli/bin/postcss'
  ]
}
ERROR 2021/01/21 14:43:13 error: failed to transform resource: exit status 1
Total in 247 ms
Error: Error building site: logged 1 error(s)
```

找不到模块，继续安装：

```shell
# 使用淘宝镜像安装
$ npm config set registry https://registry.npm.taobao.org
$ npm install postcss
```

安装后再次执行，还是报错：

```shell
$ hugo --theme=sam --buildDrafts --baseUrl="https://name.github.io/"

Building sites … Plugin Error: Cannot find module 'autoprefixer'
Require stack:
- /Users/name.blog/node_modules/postcss-cli/index.js
- /Users/name.blog/node_modules/postcss-cli/bin/postcss'
ERROR 2021/01/21 14:52:28 error: failed to transform resource: exit status 1
Total in 247 ms
Error: Error building site: logged 1 error(s)
```

继续安装：

```shell
$ npm install autoprefixer
```

泪目，终于成功了～ 🥱

```shell
                   | EN
+------------------+----+
  Pages            | 17
  Paginator pages  |  0
  Non-page files   |  0
  Static files     | 13
  Processed images |  0
  Aliases          |  0
  Sitemaps         |  1
  Cleaned          |  0

Total in 767 ms
```



