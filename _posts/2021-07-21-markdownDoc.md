---
layout: post
title: 'markdown语法手册'
subtitle: ''
date: 2021-07-21
author: Dave
cover: ''
tags: 通用文档
---

**最前面**

- [Markdown 语法教程](https://markdown.com.cn/)
    >在线查找入口
- [本文档地址](https://github.com/dendyikbc/dendyikbc.github.io/blob/master/_posts/2021-07-21-markdownDoc.md)
- 转pdf 强制分页

```xml
<div STYLE="page-break-after: always;"></div>
```

- [Markdown箭头的输入方法汇总](https://www.jianshu.com/p/d63887d0c706)
## markdown表格
```
| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |
```

| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

## markdown字体字号颜色设置
```
 <font face="钟齐吴嘉睿手写字" size=6>钟齐吴嘉睿手写字</font>

<font face="微软雅黑" size=6 color=#FF0000 >微软雅黑字体</font>

<font face="微软雅黑" >微软雅黑字体</font>

<font face="黑体" >黑体</font>

<font size=3 >3号字</font>

<font size=4 >4号字</font>

<font color=#FF0000 >红色</font>

<font color=#008000 >绿色</font>

<font color=#0000FF >蓝色</font>

<small>字体变小</small>

<big>字体变大</big>
```

 <font face="钟齐吴嘉睿手写字" size=6>钟齐吴嘉睿手写字</font>

<font face="微软雅黑" size=6 color=#FF0000 >微软雅黑字体</font>

<font face="微软雅黑" >微软雅黑字体</font>

<font face="黑体" >黑体</font>

<font size=3 >3号字</font>

<font size=4 >4号字</font>

<font color=#FF0000 >红色</font>

<font color=#008000 >绿色</font>

<font color=#0000FF >蓝色</font>

<small>字体变小</small>

<big>字体变大</big>

## 日常标注

```
- [x] 任务完成
- [ ] 任务
- [ ] 任务
```

- [x] 任务未完成
- [ ] 任务
- [ ] 任务


```
*这是斜体文字* 

_这也是斜体文字_

**这是加粗文本** 

__这也是加粗文本__

==标记文本==

~~删除文本~~
```

*这是斜体文字* 
_这也是斜体文字_

**这是加粗文本** 
__这也是加粗文本__

==标记文本==

~~删除文本~~

> 引用文本

Email邮件：
```
<123456789@qq.com>
```

<123456789@qq.com>


建立分割线

```
* * *
*****
- - -
-------------------
```

* * *
*****
- - -
-------------------

代码块

````
```c++
// 代码块
struct EndPointStr {
const char *c_str() const { return _buf; }

char _buf[sizeof("unix:") + 24];
};
```
````

```c++
// 代码块
struct EndPointStr {
    const char *c_str() const { return _buf; }

    char _buf[sizeof("unix:") + 24];
};
```

添加注解

```
这是注解,[^1] 这是[^1]另一个注解.[^pre]

[^1]: 这是1号注解

[^pre]: 这是另一个注解.
```

这是注解,[^1] 这是[^1]另一个注解.[^pre]

[^1]: 这是1号注解

[^pre]: 这是另一个注解.
