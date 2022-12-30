##### Git 全局设置

```bash
git config --global user.name "兰泱碧"
git config --global user.email "lanyangbi@joyame.com"
```

##### 创建新版本库

```bash
git clone https://gitlab.joyame.com/test/sixdu_u2.git
cd sixdu_u2
touch README.md
git add README.md
git commit -m "add README"
```

##### 已存在的文件夹

```bash
cd existing_folder
git init
git remote add origin https://gitlab.joyame.com/test/sixdu_u2.git
git add .
git commit -m "Initial commit"
```

##### 已存在的 Git 版本库

```bash
cd existing_repo
git remote rename origin old-origin
git remote add origin https://gitlab.joyame.com/test/sixdu_u2.git
```



##### 提交本地仓库后撤销

```bash
# 查看提交记录
$ git log
# 回退到指定版本
$ git reset --hard <commit-id>
```

