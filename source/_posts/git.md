title: git
date: 2016-02-21 12:57:51
tags: git
categories: tools
toc: true 
---

## 基本用法

### 日常使用的6个命令

记住最常用的6个命令就可以了，其它命令太多，用时再查。

![basic](/images/git.png)

<!-- more -->

### 创建并添加远端版本库
```bash
git init
git remote add origin https://github.com/szwchao/szwchao.github.com.git
```

### 从已有的git库中提取代码

```bash
git clone git@github.com:szwchao/szwchao.github.com.git
```

### 发布代码

```bash
git add .
git commit -m "first commit"
git push -u origin master
```

或者直接在commit后加`-a`，Git 就会自动把所有已经跟踪过的文件暂存起来一并提交

```bash
git commit -a -m 'xxx'
```

### 从远程仓库中拉取

`git pull`是`git fetch`和`git merge`命令的一个组合

```bash
git pull
```

### 撤销

如果还没有添加到暂存区，撤消对文件的修改

```bash
git checkout -- filename
```

如果已经添加了一个文件到暂存区，想从暂存区撤销，但同时在工作区里保留该文件

```bash
git reset HEAD filename
```

### 版本回退

想要撤回到上一个版本

```bash
git reset --hard commit_id
```

### 删除

从版本库里删除一个文件

```bash
git rm filename
```

### TAG

打TAG也就是发布版本

```bash
git tag -a v1.2 -m "version 1.2"
git push --tags
```

标签详情

```bash
git show v1.2
```

后期打标签

```bash
git tag -a v1.2 9fceb02
```

删除标签

```bash
git tag -d v1.2
```

## 配置

下面简单的方法可以让git diff的时候忽略换行符的差异：
```bash
git config --global core.whitespace cr-at-eol
```

配置编辑器

```bash
git config --global core.editor gvim
```

`git commit`就可以用vim来编辑commit内容了

配置diff工具

```bash
git config --global diff.tool gvimdiff

git config --global diff.tool bc3
git config --global difftool.bc3.path "c:/Program Files (x86)/Beyond Compare 3/bcomp.exe"
```

配置merge工具
```bash
git config --global merge.tool gvimdiff

git config --global merge.tool bc3
git config --global mergetool.bc3.path "c:/Program Files (x86)/Beyond Compare 3/bcomp.exe"
```

git别名

```bash
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
```

## 高级用法

#### Stash

利用stash暂时保存已做的修改

```
git stash
```

```
git stash save "说明"
```

取出暂存的修改

```
git stash pop
```

或

```
git stash pop stash@{n}
```

n为`git stash list`里的index

#### 推送本地分支并在远程新建分支

```bash
git push origin 本地分支名:远程分支名
```

#### 多个dev分支的提交合为一个提交，合并到master分支

```bash
git checkout master
git merge --squash dev
git commit –m “something from dev”
```

#### 本地忽略

文件已经被跟踪了，如果在本地想要忽略它的改动，代码库里保持原来版本并且忽略本地改动：

```bash
git update-index --assume-unchanged /path/to/file       #忽略跟踪
git update-index --no-assume-unchanged /path/to/file    #恢复跟踪
```

之后你在本地修改/path/to/file这个文件，Git也不管它了。就实现了本地忽略。

#### 打包修改过的内容

当前修改过的文件，没有commit，打包成一个压缩包：

```bash
git diff --name-only  | xargs zip update.zip
```

#### 远程撤销最近的commit

假如已经push到远程，想要回滚到上一个版本：

- 方法1：
```bash
git reset --hard HEAD~1
git push -f
```

相当于把那次错误的commit删掉了。第二步，强制推送到Github上

- 方法2：
```bash
git revert HEAD
git push
```

采取revert的方法，相当于重新生成一个提交，来撤销前一次错误的commit

#### 子模块

- 添加子模块
```bash
git submodule add 仓库地址 路径
```

- 初始化子模块并下载
```bash
git submodule update --init --recursive
```

- 当子模块有更新后，用如下命令更新
```bash
git submodule foreach git pull origin master
```

## 其它

用[这个插件](https://github.com/holygeek/git-number)使git status支持列出的文件前加上数字，然后`git number add x`即可。
