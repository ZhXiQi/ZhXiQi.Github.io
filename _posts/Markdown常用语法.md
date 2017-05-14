---
layout: post
title: Makedown常用语法
data: 2017-05-14
tag: Makedown
---
MarkDown原理：
新出的一个文档语言，使用性比较强
从后台解释语言到html。
下面是代码

# 标题
通过‘#’个数 区分h1-6（标题大小）


```
# This is an <h1> tag
## This is an <h2> tag
###### This is an <h6> tag
```
效果：
# h1>tag
## h2>tag
###### h6> tag

第二种写法：
```
Hello {#id}
---

# Hello {#id}

# Hello # {#id}
```
效果
Hello #标题
---

# Hello #id

# Hello # #id

——————

## 列表
无序代码:
```
- 文本1
    * 二级文本
- 文本2
- 文本3
```
- 文本1
    * 二级文本
- 文本2
- 文本3


有序代码
```
1. 文本1
    1. 2121
    2. 电影
2. 文本2
3. 文本3
```
1. 文本1
    1. 2121
    2. 电影
2. 文本2
3. 文本3

# 换行
空格和回车都可以

```
Here's a line for us to start with.

This line is separated from the one above by two newlines, so it will be a *separate paragraph*.
```

# 重点
```
*This text will be italic*
_This will also be italic_

**This text will be bold**
__This will also be bold__

~~This text will be crossed out.~~

_You **can** combine them_
```
*This text will be italic*
_This will also be italic_

**This text will be bold**
__This will also be bold__

~~This text will be crossed out.~~

_You **can** combine them_

# 链接
```
This is [an example](http://example.com/) inline link with a title.

[This link](http://example.net/) has no title attribute.
```
This is [an example](http://example.com/) inline link with a title.

[This link](http://example.net/) has no title attribute.


# 参考文献
这个没弄明白
```
This is [an example][id] reference-style link.
```

# 图片
```
An image: ![gras](img/image.jpg)
An image: ![gras](https://www.google.com/images/branding/googlelogo/2x/googlelogo_color_272x92dp.png)
```
An image: ![gras](https://www.google.com/images/branding/googlelogo/2x/googlelogo_color_272x92dp.png)

# 模块引用

```
As Kanye West said:

> We're living the future so
> the present is our past.
```
As Kanye West said:

> We're living the future so

> the present is our past.


# 表

```
| First Header  | Second Header |
| ------------- | ------------- |
| Content Cell  | Content Cell  |
| Content Cell  | Content Cell  |
```
| First Header  | Second Header |
| ------------- | ------------- |
| Content Cell  | Content Cell  |
| Content Cell  | Content Cell  |

# code
## 四格或缩进

```
This is a sample code block.

    Continued here.
```
This is a sample code block.

    Continued here.


# 点点点
前后加(三个点点点)
```
function test() {
console.log("notice the blank line before this function?");
}
```

# 语法高亮显示

```ruby
require 'redcarpet'
markdown = Redcarpet.new("Hello World!")
puts markdown.to_html
```

```python
import os
print('hello gitbook!')
```

# 内联代码
```
Use `gitbook` to convert the `text` in markdown
syntax to HTML.
```
Use `gitbook` to convert the `text` in markdown
syntax to HTML.

# 脚注
GitBook支持这种脚注的简单语法。脚注是相对于每个页面。
```
Text prior to footnote reference.[^2]

[^2]: Comment to include in footnote.
```
Text prior to footnote reference.[^2]

[^2]: Comment to include in footnote.


# 前端语言
GitBook支持在您的文本中使用原始HTML，不处理HTML中的Markdown语法：

```
<div>
Markdown here will not be **parsed**
</div>
```
<div>
Markdown here will not be **parsed**
</div>

# 水平规则
```
Three or more...

---

Hyphens

***

Asterisks
```
Three or more...

---

Hyphens

***

Asterisks

# 忽略
```
Let's rename \*our-new-project\* to \*our-old-project\*.
```


