title: git
date: 2015-12-21 12:57:51
tags: git
categories: tools
toc: true 
---

## 基本用法

### 日常使用的6个命令

记住最常用的6个命令就可以了，其它命令太多，用时再查。

![basic](/images/git.png)

### TAG

打TAG也就是发布版本

```bash
$ git tag -a v1.2 -m "version 1.4"
$ git push --tags
```

## 高级用法

#### 本地忽略

文件已经被跟踪了，如果在本地想要忽略它的改动，代码库里保持原来版本并且忽略本地改动：

```bash
$ git update-index --assume-unchanged /path/to/file       #忽略跟踪
$ git update-index --no-assume-unchanged /path/to/file    #恢复跟踪
```

之后你在本地修改/path/to/file这个文件，Git也不管它了。就实现了本地忽略。
