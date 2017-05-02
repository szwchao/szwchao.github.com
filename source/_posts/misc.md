title: misc
date: 2015-12-25
tags: [other]
categories: other
toc: true 
---

## pandoc

转docx

```
pandoc -s -S example.md -o example.docx
```

有模板的docx

```
pandoc -s -S example.md --reference-docx template.docx -o example.docx
```

加css样式

```
-c github.css
```

把css样式固化在html里

```
pandoc foo.md -s -c github.css --self-contained -o foo.html
```

## misc

输出所有字体

```
fc-list.exe :lang=zh-cn > cn.txt
```
