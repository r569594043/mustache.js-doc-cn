#mustache — 轻逻辑模版语言。

## 大纲(SYNOPSIS)

一个典型的Mustache模版实例：

Hello {{name}}
You have just won ${{value}}!
{{#in_ca}}
Well, ${{taxed_value}}, after taxes.
{{/in_ca}}
绑定以下键值的集合：

	{
		"name": "Chris",
		"value": 10000,
		"taxed_value": 10000 - (10000 * 0.4),
		"in_ca": true
	}
会生成:

Hello Chris
You have just won $10000!
Well, $6000.0, after taxes.

## 简介(DESCRIPTION)

[Mustache](http://mustache.github.com/) 可以用于HTML，配置文件、程序代码等，核心机制就是把模版中的tags替换为通过hash or object给定的值。

我们称之为“轻逻辑”是因为模版语言中没有if、else 和 for循环，只有tags。一些tags会被替换为一个值，还有一些会被替换为一列值。本文会介绍Mustache不同种类的tags.

## TAG的种类(TAG TYPES)

Tags用双花括号标记，`{{person}}`就是一个tag，`{{#person}}`也是。在所有示例中，我们都将用`person`作为属性名或tag的名称。下面开始介绍不通种类的tags.

### 变量(Variables)

最基础的tag就是变量。模版中的tag`{{name}}`会在当前上下文中匹配名为`name`的值。如果没有找到，就不会输出。

所有值的输出默认是经过HTML编码的。如果想输出原文，可以使用三重花括号的标记：`{{{name}}}`。

也可以使用 `&` 来标记原文输出：{{& name}}。这种方式在changing delimiters的时候可能有用(see “Set Delimiter” below)。

如果匹配某个值失败会返回一个空字符串。Ruby版本的Mustache在这种情况下可以抛出异常，例如：

模版：

	* {{name}}
	* {{age}}
	* {{company}}
	* {{{company}}}

键值：

	{
		"name": "Chris",
		"company": "<b>GitHub</b>"
	}

输出：

	* Chris
	*
	* &lt;b&gt;GitHub&lt;/b&gt;
	* <strong>GitHub</strong>

### Sections

Sections可以根据当前上下文中的键值来渲染文本块一次或更多次，

Sections以“#”开始、以“/”结束，比如{{#person}}…{{/person}}。

Sections的行为由给定的键值决定。

#### False 或 空列表

如果名为`person`的值为false或为空，Sections将不会输出任何字符。

模版：

	Shown.
	{{#nothin}}
	Never shown!
	{{/nothin}}

键值：

	{
	  "person": true,
	}

输出：

	Shown.

#### 非空的列表：

如果名为`person`的值不为false，Sections内部的字符将会输出一次或多次。

当值是为空的列表时，与列表中的每个条目对应的字符都会显示出来。每次迭代的时候上下文会从区块切换到对应的每一个条目，这样我们就可以遍历一个集合了。

模版：

	{{#repo}}
	<b>{{name}}</b>
	{{/repo}}

键值：

	{
		"repo": [
			{ "name": "resque" },
			{ "name": "hub" },
			{ "name": "rip" }
		]
	}

输出：

	<b>resque</b>
	<b>hub</b>
	<b>rip</b>

#### 匿名函数

当给定的值为object，例如 「function」 或 「匿名函数」，object会被调用并传进区块。被传递进区块的是未经编译的字符串。此时{{tags}}还没有被替换，用这种方式我们可以实现过滤和缓存。

模版：

	{{#wrapped}}
		{{name}} is awesome.
	{{/wrapped}}

键值：

	{
		"name": "Willy",
		"wrapped": function() {
			return function(text) {
				return "<b>" + render(text) + "</b>"
			}
		}
	}

输出：

	<strong>Willy is awesome.</strong>

#### 非False的值

一个不是列表并且不为空的值，会被用来进行单一的渲染。

模版：

	{{#person?}}
	Hi {{name}}!
	{{/person?}}

键值：

{
  "person?": { "name": "Jon" }
}
输出：

Hi Jon!

### 反向的Sections(Inverted Sections)

反向的区块以「^」开始、以「/」结束，比如 {{^person}}…{{/person}}

当给定的值不存在、为空或false的时候，反向的区块会被渲染。

模版：

	{{#repo}}<b>{{name}}</b>{{/repo}}
	{{^repo}}No repos :({{/repo}}

键值：

	{
	  "repo": []
	}

输出：

	No repos :(

### 注释

以「!」开始的注释文字将会被忽略：

	<h1>Today{{! ignore me }}.</h1>

会被渲染成：

	<h1>Today.</h1>

注释可以换行。

### 子模版

子模版以「>」标记开始，比如 {{> box}}

子模版的渲染是即时的，所以对子模版的递归是可行的，避免了无尽的循环。

子模版还会集成调用的长下文，在 ERB 你可能见过这种code:

	<%= partial :next_more, :start => start, :size => size %>

Mustache 只需要:

	{{> next_more}}

因为`next_more.mustache` 会继承调用时上下文的`size`和`start`方法

也许你会发现子模版和 includes、template扩展很像，尽管和字面上的描述不太一致。

例如，如下的模版和子模版：

	base.mustache:
	<h2>Names</h2>
	{{#names}}
		{{> user}}
	{{/names}}

	user.mustache:
	<strong>{{name}}</strong>

可以认为看成一个合成的模版：

	<h2>Names</h2>
	{{#names}}
		<strong>{{name}}</strong>
	{{/names}}

### 设置分隔符

以「=」开始的tag 可以标记 `{{` 和 `}}` 中间的字符为自定义字符。

引自 [ctemplates](http://google-ctemplate.googlecode.com/svn/trunk/doc/howto.html), this “is useful for languages like TeX, where double-braces may occur in the text and are awkward to use for markup.”

自定义字符不能包含空格和等号。

## COPYRIGHT

Mustache is Copyright (C) 2009 Chris Wanstrath

Original CTemplate by Google
