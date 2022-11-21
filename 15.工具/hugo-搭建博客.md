> Hugo is a fast and modern static site generator written in Go, and designed to make website creation fun again.
>
> Hugo 是使用 Go 语言编写的 [静态网站生成器](https://gohugo.io/about/what-is-hugo/)，页面编译速度快，支持跨平台、皮肤自定义，方便部署。



[搭建教程](https://gohugo.io/getting-started/quick-start/)，官方教程写的非常详细，下文是我自己的搭建记录。 👀



## 1. 安装

使用 Mac Homebrew 进行安装

```shell
# 跳过 brew 检查更新（可以忽略）
$ export HOMEBREW_NO_AUTO_UPDATE=true

# 开始安装
$ brew install hugo

# 出现下面内容就是安装完成了
==> Summary
🍺  /usr/local/Cellar/hugo/0.55.0: 31 files, 40.6MB
```



## 2. 建站

创建一个自定义路径，执行命令

```shell
$ hugo new site $YourBlogName
```

这样就建立了自己的站点，目录接口如下：

```shell
.
├── archetypes # 存放生成博客的模版
├── assets     # 存放被 Hugo Pipes 处理的文件
├── config     # 配置文件；支持 JSON YAML TOML 三种格式配置文件
├── content    # 存放博客 markdown 文件
├── data       # 存放 Hugo 处理的数据
├── layouts    # 存放布局文件
├── static     # 存放静态文件 图片 CSS JS文件
└── themes     # 存放主题
```

目前无法使用，还需要设置网站主题。



## 3. 添加主题

**[主题库](https://themes.gohugo.io/) 可供挑选，以 Sam 为例，下载方法：**

```shell
$ cd $YourBlogName
$ git init
$ git clone https://github.com/victoriadrake/hugo-theme-sam.git themes/sam
# 选好主题后注意查看最低支持的 hugo 版本
```

**配置站点**

将 `exampleSite/config.toml` 复制替换 `$YourBlogName/config.toml` 即可，在 `config.toml` 文件内进行基础设置， Sam 主题，可以在配置内添加主页的主题背景图/视频/背景色。

视频/图片存放路径：`themes/sam/static/`

```python
[[params.videoBackground.sources]]
    source  = "sample_video.mp4"
    type    = "video/mp4"
    poster  = "background.png"
[params.videoBackground]
    overlay = "rgba(0, 0, 0, 0.4)" # 这是背景色+透明度

# 设置分类及链接
[[params.mainMenu]]  
    link = "test_summary"
    text = "Test Summary"

[[params.mainMenu]]
    link = "python"
    text = "Python"

[[params.mainMenu]]
    link = "essay"
    text = "Essay"

[[params.mainMenu]]
    link = "about/about/"
    text = "Who am I ?"

```



## 4. 访问

命令行启动

```shell
$ hugo sever -D
```

在浏览器中访问 [localhost:1313](localhost:1313) ，刷新页面可以实时预览。



## 5. 写博客

创建文章

```shell
$ hugo new about/about.md
```

该命令就是在 `/content/` 目录下创建了 `about` 目录，该目录内包含一个 `about.md` 的文件，这个文件就是第一篇博客的名称。可以通过 `http://localhost:1313/about/about/` 访问。

其实这个 `about` 目录就是上面我配置文件内最后一个菜单的关联目录，通过点击主页的 `Who am I ?` 也可以实现跳转。

同理，可以在 `content` 目录下根据自己的喜好，建立不同的分类，来进行文章管理。

---

选择其他主题也大同小异...



