由于项目的复杂度越来越高,前端工程化越来越重要,前端工程化大致为模块化,组件化,规范化,自动化.

今天主要学习规范化中的CSS的规范问题.,项目庞大,需要对CSS文件进行很好的分类.

这里主要说单页应用.

### 目录结构

```
## 建议在styles目录下写个readMe.md文档.解释各个文件的作用
## 目录
|-- styles
	|-- animation  (动画)
		|-- index.scss
		|-- fade.scss
		|-- ease.scss
		|-- slide.scss
		|-- ...  #etc..

	|-- common  (全局样式)
		|-- index.scss #导入该文件下需要的scss模块
		|-- base.scss # 主要是reset和其他一些通用的css
		|-- normalize.scss
		|-- ...        # 根据自己的实际项目添加其他scss文件.如排版/模板/通用的全局变量抽取为一个scss.

	|-- layout.scss
		|-- index.scss
		|-- grid.scss     #grid system
		|-- header.scss   # 通用的header
		|-- footer.scss   # 通用的footer
		|--  ...          # etc.. 其他通用的布局scss

	|-- components  (组件样式(通用组件/业务组件))
		|-- index.scss
		|-- 各个组件的scss.按照组件名进行命名
			#每个scss文件需要在文件开头保持良好的注释习惯.在项目中也可以根据实际情况制定一套通用的注释规范

	|-- pages (具体页面)
		|-- 针对一些页面写的特定的样式
		|-- index.scss

	|-- mixins  (混入)
		|-- index.scss
		| xxx(按照功能名).scss

	|-- themes (皮肤样式)
		|-- index.scss
		|-- default(默认主题)
			|-- ...
		|-- xxx(定制主题)
			|-- ...
	|-- vendors (包含来自外部的库和框架的CSS文件)
		-- inex.scss
		-- ...

	|-- index.scss   # primary Scss file
```

### css命名规则

CSS最难的难在起名字.

常见的有[BEM风格](http://getbem.com/naming/),[Bootstrap风格](http://getbootstrap.com/)

推荐阅读.[如何看待 CSS 中 BEM 的命名方式？](https://www.zhihu.com/question/21935157)

```
对于人数少的团队也许比较随意点,但是最好要定义一些基本的规范.
1. 语义化
2. 使用"-"连字符还是驼峰
```

```
我们在编写自己的业务组件时候,最外层一个div#id(组件名).
通过这种方式杜绝命名污染问题.缺点:ID在一个页面中的唯一性导致了如果以ID为选择器来写CSS，就无法重用。
```

### 代码格式

一些基本的代码格式

```
1. 引号的使用:单引号还是双引号
2. 简短的css单行定义完.不换行.复杂的通过嵌套定义
3. 值为0时的单位是否省略
4. 颜色值的表示方法的统一.
5. 根据属性的重要性,浏览器渲染性能按顺序书写
```

```
1.位置属性(position, z-index,
display, float,top, right等)
2.大小(width, height, padding, margin ...)
3.文字系列(font, line-height, letter-spacing,
color- text-align等)
4.背景(background, border等)
5.其他(animation, transition等)
```
CSS渲染方面的性能可以了解下浏览器的工作机制.比如重绘/重排.

或者参考NEC的图:
![](http://img.blog.csdn.net/20170513170230244?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveGlhb3pvbmdnZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### CSS的性能优化

我就不做搬运工了.
推荐阅读[CSS性能优化的方法有哪些](https://www.zhihu.com/question/19886806)

### 可以定义一套自己内部使用的通用命名



```
文档	doc
头部	header
主体	body
尾部	footer
主栏	main
主栏子容器
侧栏	side
侧栏子容器 sidec
盒容器	wrap
导航	nav
子导航	subnav
面包屑	crumb
菜单	menu
选项卡	tab
标题区	head/title
内容区	body/content
列表	list
表格	table
表单	form
热点	hot
排行	top
登录	login
标志	logo
广告	advertise
搜索	search
幻灯	slide
提示	tips
帮助	help
新闻	news
下载	download
注册	regist
投票	vote
版权	copyright
结果	result
标题	title
按钮	button
输入	input
选中	selected
当前	current
显示	show
隐藏	hide
打开	open
关闭	close
出错	error
不可用disabled
```
