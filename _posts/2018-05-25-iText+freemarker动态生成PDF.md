---
layout: post
title: iText+Freemarker动态生成PDF模板
data: 2018-05-25
tag: iText、Freemarker、JFreeChart
---

# iText+Freemarker动态生成PDF模板

    本次分享主要包含三个部分：iText部分、freemarker模板部分以及JFreeChart绘制图标部分
    
![iText&Freemarker](/images/posts/iText+Freemarker+JFreeChart/iText+Freemarker.jpg)

### 1、iText：

iText是著名的开放源码的站点sourceforge一个项目，是用于生成PDF文档的一个java类库。通过iText不仅可以生成PDF或rtf的文档，而且可以将XML、Html等文件转化为PDF文件。 iText的安装非常方便，下载iText.jar文件后，只需要在系统的CLASSPATH中加入iText.jar的路径，在程序中就可以使用iText类库了。

![iText](/images/posts/iText+Freemarker+JFreeChart/iText.jpg)

包：com.itextpdf.text

主要使用到和常见的对象：

* Document：是用于生成PDF文档的主要类。这是第一个需要实例化的类。创建文档之后，需要一个编写器来向其中写入信息。

* PdfWriter是一个PDF编写器。

* Paragraph：此类表示一个缩进的段落

* Chapter：此类表示PDF文档中的章，其使用Paragraph作为标题，int作为章节编号来创建该类

* Font：此类包含一种字体的所有规范，比如字体集、字号、样式和颜色。各种字体都在此类中声明为静态常量

* List：iText API中也有一个List类，这里的List表示一个列表，该列表又包含许多的ListItems。相当于是专为生成PDF所封装的一个list

* PDFTable：这表示一个表格，可以使用绝对位置来放置它，当然也可以作为类Table添加到Document文档中

* Anchor：一个Anchor可能是一个引用，或者是一个引用的目标，也就是实现点击一个地方，跳转到所指定的位置。这个目前需求倒是没有用到

* Chunk(块)：块是能被添加到文档的文本的最小单位，块可以用于构建基础元素，如：短句、段落、锚点等，块是一个有确定字体的字符串，要添加块到文档中时，其他所有布局变量均要被定义

### iText的需求分析

一般情况下，iText使用在有以下一个要求的项目中：

* 内容无法提前利用：取决于用户的输入或实时的数据库信息。

* 由于内容，页面过多，PDF文档不能手动生成。

* 文档需在无人参与，批处理模式下自动创建。

* 内容被定制或个性化；例如，终端客户的名字需要标记在大量的页面上。

[这里](http://rensanning.iteye.com/blog/1538689)是一个不错的学习教程!!

### 2、freemarker

FreeMarker是一款模板引擎：即一种基于模板和要改变的数据，并用来生成输出文本（HTML网页、电子邮件、配置文件、源代码等）的通用工具。它不是面向最终用户的，而是一个Java类库，是一款程序员可以嵌入他们所开发产品的组件。

FreeMarker是免费的，基于Apache许可证2.0版本发布。其模板编写为FreeMarker Template Language（FTL），属于简单、专用的语言。需要准备数据在真实编程语言中来显示，比如数据库查询和业务运算，之后模板显示已经准备好的数据。在模板中，主要用于如何展现数据，	而在模板之外注意于要展示什么数据.

freemarker模板语法类似HTML的ftl文件，但是其与纯HTML还是有很大的区别的，比如ftl文件无法使用内置的js函数，不支持复杂的css布局等，但是ftl有他自己的內建函数库，可以满足大部分的需求

![ftl](/images/posts/iText+Freemarker+JFreeChart/ftl.jpg)
![freemarker](/images/posts/iText+Freemarker+JFreeChart/freemarker.png)

详情可访问[这里](http://www.kerneler.com/freemarker2.3.23/ref_builtins_string.html)

### 3、JFreeChart

JFreeChart 是开放源代码站点 SourceForge.net 上的一个 JAVA 项目，它主要用来各种各样的图表，这些图表包括：饼图、柱状图 ( 普通柱状图以及堆栈柱状图 )、线图、区域图、分布图、混合图、甘特图以及一些仪表盘等等。这些不同式样的图表基本上可以满足目前的要求。

![JFreeChart](/images/posts/iText+Freemarker+JFreeChart/JFreeChart.png)

JFreeChart中几个核心的对象类：

| 类名 | 类的作用以及简单描述 |
| -- | -- |
| JFreeChart | 图表对象，任何类型的图表的最终表现形式都是在该对象进行一些属性的定制。JFreeChart 引擎本身提供了一个工厂类用于创建不同类型的图表对象 |
| XXXXXDataset | 数据集对象，用于提供显示图表所用的数据。根据不同类型的图表对应着很多类型的数据集对象类 |
| XXXXXPlot | 图表区域对象，基本上这个对象决定着什么样式的图表，创建该对象的时候需要 Axis、Renderer 以及数据集对象的支持 |
| XXXXXAxis | 用于处理图表的两个轴：纵轴和横轴 |
| XXXXXRenderer | 负责如何显示一个图表对象 |
| XXXXXURLGenerator | 用于生成 Web 图表中每个项目的鼠标点击链接 |
| XXXXXToolTipGenerator | 用于生成图象的帮助提示，不同类型图表对应不同类型的工具提示类 |

详情可访问[这里](https://www.ibm.com/developerworks/cn/java/l-jfreechart/index.html)