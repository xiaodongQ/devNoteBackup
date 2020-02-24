## hugo博客搭建

* 使用背景
	- 当前博客
		+ fork自别人现有的博客改了点描述信息，用的是`Jekyll`
			* `Jekyll`基于Ruby的静态网页生成系统，采用模板将Markdown(或Textile)文件转换为统一的网页GitHub Pages 免费的静态站点
		+ [git博客](https://github.com/xiaodongQ/xiaodongq.github.io)
	- Hugo是一个用Go语言编写的静态网站生成器，Hugo生成静态页面的效率很高。[使用hugo搭建个人博客站点](https://blog.coderzh.com/2015/08/29/hugo/)
    - 发现换个生成器，并没有什么使用上的变化，博客文章中的语法标识不一样而已。。。至于hugo代码的学习先搁置了
* [Hugo中文文档](https://www.gohugo.org)
	- `hugo server --theme=hugo-journal --buildDrafts`
* 过程问题
	- 使用`ConvertToHugo.py`或`JekyllToHugo`将Jekyll的post文章转换为hugo，[Jekyll](https://www.gohugo.org/doc/tools/#migration-tools)，报包导入失败：`ImportError: No module named yam`。解决：`pip install pyyaml`
