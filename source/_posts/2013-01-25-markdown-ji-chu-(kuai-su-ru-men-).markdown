---
layout: post
title: "Markdown: 基础 （快速入门）"
date: 2013-01-25 13:06:26 +0800
comments: true
categories: 
---
Markdown 其实很简单就可以上手，此文档提供了一些使用范例，用作备忘。
<!-- more -->
##段落、标题、区块代码##

一个段落是由一个以上的连接的行句组成，而一个以上的空行则会划分出不同的段落（空行的定义是显示上看起来像是空行，就被视为空行，例如有一行只有空白和 tab，那该行也会被视为空行），一般的段落不需要用空白或换行缩进。

单个回车视为空格。连续回车才能分段。行尾加两个空格，即可段内换行。

显示如下代码框只需要在文字前面添加四个空格或者是一个`<tab>`即可

	显示当前代码框只需要在文字前面添加四个空格或者是一个tab即可

Markdown 支持两种标题的语法，Setext 和 atx 形式。  
Setext 形式是用底线的形式，利用 = （最高阶标题）和 - （第二阶标题）。  

	第一阶标题
	===========

第一阶标题
===========

	第二阶标题
	----------

第二阶标题
----------

Atx 形式在行首插入 1 到 6 个 # ，对应到标题 1 到 6 阶。

	#第一阶标题  
#第一阶标题

	##第二阶标题  
##第二阶标题

	###第三阶标题  
###第三阶标题  
……

	######第六阶标题  
######第六阶标题

区块引用则使用 email 形式的 '>' 角括号。

	> 此处是区块引用  
> 此处是区块引用

##修辞和强调

Markdown 使用星号和底线来标记需要强调的区段。

	*首尾各一个*号这些文字显示为斜体*  
*首尾各一个星号这些文字显示为斜体*

	**首尾各两个*号这些文字显示为粗体**  
**首尾各两个星号这些文字显示为粗体**

	***首尾各三个*号这些文字显示为斜体加粗体***  
***首尾各三个星号这些文字显示为斜体加粗体***

	_首尾使用底线强调区段_  
_首尾使用底线强调区段_

##列表

无序列表使用星号、加号和减号来做为列表的项目标记，这些符号是都可以使用的.

星号：

	* Candy.
	* Gum.
	* Booze.  
* Candy.
* Gum.
* Booze.

加号：

	+ Candy.
	+ Gum.
	+ Booze.  
+ Candy.
+ Gum.
+ Booze.

和减号：

	- Candy.
	- Gum.
	- Booze.  
- Candy.
- Gum.
- Booze.

有序的列表则是使用一般的数字接着一个英文句点作为项目标记：

	1. Red
	2. Green
	3. Blue  
1. Red
2. Green
3. Blue

	如果你在项目之间插入空行，那项目的内容会用 <p> 包起来，你也可以在一个项目内放上多个段落，只要在它前面缩排 4 个空白或 1 个 tab 。  
如果你在项目之间插入空行，那项目的内容会用 <p> 包起来，你也可以在一个项目内放上多个段落，只要在它前面缩排 4 个空白或 1 个 tab 。

---
上面是一条横线  

	---
	上面是一条横线

两个列表之间不能相邻，否则会解释为嵌套的列表

	- 外层列表项目
	 + 内层列表项目
	 + 内层列表项目
	 + 内层列表项目
	- 外层列表  
- 外层列表项目
 + 内层列表项目
 + 内层列表项目
 + 内层列表项目
- 外层列表

项目Markdown 支援两种形式的链接语法： 行内 和 参考 两种形式，两种都是使用角括号来把文字转成连结。

行内形式是直接在后面用括号直接接上链接：

	This is an [example link](http://example.com/).

This is an [example link](http://example.com/).

你也可以选择性的加上 title 属性

	This is an [example link](http://example.com/ "With a Title").

This is an [example link](http://example.com/ "With a Title").

参考形式的链接让你可以为链接定一个名称，之后你可以在文件的其他地方定义该链接的内容：

	I get 10 times more traffic from [Google][1] than from
	[Yahoo][2] or [MSN][3].

	[1]: http://google.com/ "Google"
	[2]: http://search.yahoo.com/ "Yahoo Search"
	[3]: http://search.msn.com/ "MSN Search"

I get 10 times more traffic from [Google][1] than from
[Yahoo][2] or [MSN][3].

[1]: http://google.com/ "Google"
[2]: http://search.yahoo.com/ "Yahoo Search"
[3]: http://search.msn.com/ "MSN Search"

title 属性是选择性的，链接名称可以用字母、数字和空格，但是不分大小写：

	I start my morning with a cup of coffee and
	[The New York Times][NY Times].

	[ny times]: http://www.nytimes.com/

I start my morning with a cup of coffee and
[The New York Times][NY Times].

[ny times]: http://www.nytimes.com/

##图片

图片的语法和链接很像。

行内形式（title 是选择性的）：

	![alt text](/path/to/img.jpg "Title")

参考形式：

	![alt text][id]

	[id]: /path/to/img.jpg "Title"

##代码

在一般的段落文字中，你可以使用反引号 ` 来标记代码区段，区段内的 &、< 和 > 都会被自动的转换成 HTML 实体，这项特性让你可以很容易的在代码区段内插入 HTML 码：

	I strongly recommend against using any `<blink>` tags.

	I wish SmartyPants used named entities like `&mdash;`
	instead of decimal-encoded entites like `&#8212;`.  
I strongly recommend against using any `<blink>` tags.

I wish SmartyPants used named entities like `&mdash;`
instead of decimal-encoded entites like `&#8212;`.

如果要建立一个已经格式化好的代码区块，只要每行都缩进 4 个空格或是一个 tab 就可以了，而 &、< 和 > 也一样会自动转成 HTML 实体。

Markdown 语法:

	If you want your page to validate under XHTML 1.0 Strict,
	you've got to put paragraph tags in your blockquotes:

	<blockquote>
	<p>For example.</p>
	</blockquote>  
If you want your page to validate under XHTML 1.0 Strict,
you've got to put paragraph tags in your blockquotes:

<blockquote>
<p>For example.</p>
</blockquote>
