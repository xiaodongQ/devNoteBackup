## obsidian

### 快捷键

几个建议的快捷键：

在资源管理器中打开，ctrl+shift+r （mac：command+shift+r）
在文件列表中打开，ctrl+shift+e 

### 推荐主题

几个还不错的主题
* Things，日常用的，黑色

可以让AI修改各级标题颜色。比如当前主题下，一级二级是白色的，我就调整为了  
```sh
# theme.css
37 +  --h1-color: #e87d3e;
38 +  --h2-color: #e5b567;
```

* Blue Topaz
* Obsidian Nord
* Shimmering Focus

### 插件

> 可见：[社区插件](https://pkmer.cn/Pkmer-Docs/10-obsidian/obsidian%E7%A4%BE%E5%8C%BA%E6%8F%92%E4%BB%B6/obsidian%E7%A4%BE%E5%8C%BA%E6%8F%92%E4%BB%B6/)

到github下载，基本都是`main.js`、`manifest.json`、`styles.css`三个文件，到`.obsidian/plugins`目录新增插件名对应目录，拷贝进去而后启用。

* data-files-editor：增加了在黑曜石中创建和编辑.txt, .json, .xml 类型文件的能力
* number-headings-obsidian：给笔记中的标题自动编号，以及动态目录
* open tab setting：自定义 Obsidian 打开选项卡和在文件之间导航的方式，在新标签页打开文件以及避免重复标签页
* excalidraw 画图
    * data.json里可修改默认存放位置，比如`"folder": "01_sources/_Excalidraw",`
    * 默认保存的文件名格式修改：保存->文件名
* drawio 画图
* claudian，打开claude code
    * terminal，打开终端（在mac上还可以，开启claude code，在windows上就不好用了）
* control characters，展示空格和tab

vim插件：配置文件为`.obsidian.vimrc`（vault仓库那级，而不是`.obsidian`目录），禁用vim模式下的复制粘贴等键，避免影响windows下的`ctrl+c/v`等快捷键功能。

```sh
" vim模式下禁用Ctrl-A、Ctrl-V、Ctrl-c、Ctrl-x
unmap <C-a>
unmap <C-x>
unmap <C-c>
unmap <C-v>
unmap v
```

另外，建议取消ctrl+enter快捷键，否则占用了跳到下一行的功能

### 浏览器插件

* obsidian web clipper：在浏览器快捷保存内容到Obsidian知识库
    * default里面的Note location可以设置保存路径
    * `cmd + shift + o`快捷键，保存页面内容到obsidian