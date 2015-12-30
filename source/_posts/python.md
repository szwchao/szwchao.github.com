title: python
date: 2015-12-21 14:52:04
tags: python
categories: code
toc: true 
---
# Python
## 常用数据类型

### list

list是一种有序的集合，可以随时添加和删除其中的元素。

列表常用方法：

* 用len()函数可以获得list元素的个数
* 用索引来访问list中每一个位置的元素： 第一个`a[0]`，倒数第一个`a[-1]`
* 追加元素到末尾：`a.append('xxx')`
* 插入到指定的位置，比如索引号为1的位置：`a.insert(1, 'yyy')`
* 删除list末尾的元素：`a.pop()`，或者删除指定位置的元素：`a.pop(1)`

<!-- more -->

### tuple

元组和列表类似，但是tuple一旦初始化就不能修改。

但如果像下面的例子：tuple里的list可以改变。原因是第三个其实是指向list的一个元素，而该list指向`['A', 'B']`，指向的list不可以变，但list指向的值可以变

```python
>>> t = ('a', 'b', ['A', 'B'])
>>> t[2][0] = 'X'
>>> t[2][1] = 'Y'
>>> t
('a', 'b', ['X', 'Y'])
```

### dict

字典是key-value哈希的，所以key不能有重复，并且必须是字符串，整数等不可变的可以哈希的类型，list就不能作为key。

字典常用方法：

* 赋值： `d['a'] = 1`
* 取值： `d['a']`
    * 如果键值不存在，会抛出异常：KeyError。用`in`判断键值，或者用`d.get('a', 0)`，如果key不存在，返回指定value。
* 删除一个key： `d.pop('a')`

## tips
#### Python测试程序运行时间
 
```python
import time
start = time.time()
run_fun()
end = time.time()
print end-start
```

#### 正则

正则表达式使用反斜杠" \ "来代表特殊形式或用作转义字符，这里跟Python的语法冲突，因此，Python用" \\\\ "表示正则表达式中的" \ "，因为正则表达式中如果要匹配" \ "，需要用\来转义，变成" \\ "，而Python语法中又需要对字符串中每一个\进行转义，所以就变成了" \\\\ "。
