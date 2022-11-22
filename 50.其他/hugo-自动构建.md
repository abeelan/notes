通过 sh 脚本完成一键构建，更新博客。



### 创建帖子

```shell
$ hugo new posts/blog-自动构建.md
```



### 重新构建 hugo 页面

编辑完成帖子后，更新网站。

```bash
#!/bin/sh
# deploy.sh

hugo --theme=dream --buildDrafts --baseUrl="https://abeelan.github.io/"

echo "\033[0;32mDeploying updates to GitHub...\033[0m"

# Go To Public folder
cd public

# Add changes to git.
git add .

# Commit changes.
msg="rebuilding site `date`"
if [ $# -eq 1 ]
  then msg="$1"
fi
git commit -m "$msg"

# Push source and build repos.
git push origin main

# Come Back up to the Project Root
cd ..

```

在项目目录下执行上面脚本，记得添加一下可执行权限。

```shell
$ ./deploy.sh
```

