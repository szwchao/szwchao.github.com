title: vim
date: 2015-12-25 10:47:03
tags: [tools, vim]
categories: tools
toc: true 
---

## 变量

几种不同的变量：

* b:name - buffer缓冲区的变量
* w:name - window变量
* g:name - 全局变量
* v:name - Vim预定义变量
* s:name - 作用域是当前脚本的变量
 
Vim中有一些特殊变量，包括：

* % - 当前文件名
* %< - 当前文件名，去后缀
* <cword> - 光标所在的单词

<!-- more -->

## 表达式

数值、字符串都是表达式，其他常见的表达式包括：

* $name - 环境变量名
* &name - Vim选项名
* @r - Vim寄存器名


## 同时对几个文件操作
 
```vim
 :args *.c
 :argdo set ff=unix | update
 :args *.[ch]
 :argdo %s/\<my_foo\>/My_Foo/ge | update
```

## 在Visual模式里替换

```
music amuse fuse refuse
```

替换us为az，先用Visual选择模式或列模式里选中要替换的文本，然后ESC回到normal模式，再执行

```
:s/\%Vus/az/g
```

## 配色

* 列出所有rgb.txt里的配色
自己写的命令 `DisplayAllColors`

* 列出当前配色
`so $VIMRUNTIME/syntax/hitest.vim`

* 列出高亮组最近在哪里设置
`verbose hi Comment`

## 正则

* 文本里

* 脚本里
 
`matchstr({expr}, {pat}[, {start}[, {count}]])			*matchstr()*`
