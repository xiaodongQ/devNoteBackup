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
* obsidian-opener：默认在新标签页中打开 md 笔记，如果已经打开了该 md，就会直接切换到该标签页
* excalidraw 画图
* drawio 画图
* claudian，打开claude code
    * terminal，打开终端（在mac上还可以，开启claude code，在windows上就不好用了）
* control characters，展示空格和tab
