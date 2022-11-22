记录下 Mac 上的 SVN 命令行使用，脑子不好总是忘记。



## 安装

```shell
$ brew install svn
Error:
  homebrew-core is a shallow clone.
  homebrew-cask is a shallow clone.
To `brew update`, first run:
  git -C /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core fetch --unshallow
  git -C /usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask fetch --unshallow
```

出现报错，按照提示 git 合并下分支

```shell
$ brew install svn
Error: python@3.9: wrong number of arguments (given 1, expected 0)
```

出现报错，手动更新下。

```shell
$ brew update
Already up-to-date.
```

再次安装

```shell
$ brew install svn
...
```

## 常用命令

下载仓库代码（checkout）

```shell
$ svn co {project_link}
```

