**warp终端**

目前Windows下在用WezTerm（也是Rust开发），配置了一些自定义功能和快捷键，用得挺顺手。不过有个点，右键（注册表里增加的）在某个目录打开wezterm终端，然后开Claude Code或其他CLI，打开多了就会有多个终端实例，不好统一管理。

warp终端能比较好解决这个问题。

仓库地址：[warpdotdev/warp](https://github.com/warpdotdev/warp)

## 1. 快捷键

### 1.1. 面板：
* `cmd + /` 可调出**各快捷键说明面板**，可以按功能搜索
* `cmd + ,` 可调出设置面板

### 1.2. 展示栏：
* `cmd + B`，打开/隐藏 左侧tab栏（展示各个打开的tab页）
* `cmd + \`，打开/隐藏 目录浏览器（显示当前目录 和 已打开文件所在的目录）
* `cmd + T`，创建新tab，默认创建和当前tab一样的tab（不过不会继承自定义的标题和颜色），可以在`Features->Session`里设置目录行为
* `ctrl + T`，切换tab，`Features->Keys`里可以指定循环切换还是最近的tab切换（我改成了最近的tab切换）

### 1.3. 分屏：

* `cmd + D`，向右侧分屏
* `cmd + shift + D`，向下分屏

分屏后，左侧的tab还是复用的，不会增加一个新的

### 1.4. 选择
* `cmd + 上下键`，选择一块，**很实用**（每次回车或者重新输命令，都是新的块/block，可以按块复制）

### 1.5. 工作流：
* `ctrl + shift + R` 调出工作流（workflow）

1）默认预定义了一些快捷命令，可选择使用。比如docker命令、git命令等

比如选择 `docker exec -it container_name bash` 后，光标会停在`container_name`占位上，只要填对应容器名就可以快速进入容器。

2）也可以定义自己的工作流，`cmd + P`调出命令搜索面板，输入`create new workflow`即可开始创建。具体见 [warp-drive/workflows文档](https://docs.warp.dev/knowledge-and-collaboration/warp-drive/workflows)

## 2. 使用技巧

### 2.1. 自定义设置快捷tab入口
1、可以自定义设置快捷tab入口，指定标题、颜色、打开时的默认目录、要执行什么命令

tab配置文件所在目录 `~/.warp/tab_configs/`
* 颜色支持：`red`、`green`、`yellow`、`blue`、`magenta`品红色、`cyan`青色

示例：

```sh
# 点击"+"时显示的名称
name = "知识库目录"
# 创建后左侧tab显示的标题
title = "知识库目录"
# tab显示的颜色s
color = "yellow"

[[panes]]
id = "main"
type = "terminal"
directory = "/Users/xd/Documents/note/obsidian_xd"
commands = []
```

### 2.2. 表现设置

UI页面可以进行各类设置。
* `Features->Session`里，可设置历史保留行数、创建新Session的目录行为等
* `Features->Keys`里，可设置`ctrl+T`行为，上面快捷键里已说明了
* `Features->Terminal Input`里，可自定义各类输入行为，比如选中即复制（默认就是这样）

对默认做了一些调整，主要涉及显示样式，配置文件在：`~/.warp/settings.toml`。可查看下面“我的配置文件”

### 2.3. VIM操作

`Settings > Features > Text Editing`中，启用`“Edit commands with Vim keybindings”`

### 2.4. alias 快捷行为

终端输入`alias`，会展示在`~/.zshrc`里自己设置的alias，**以及终端自带的一些快捷命令**

### 2.5. 右键行为

1、warp终端里，每次命令+结果，都是一个 `块`/`block`，点击是按block为粒度，可以`cmd+c`（mac下）复制整个块；可以`cmd+f`块内搜索

2、右键后，也提供了快捷的 复制当前目录、复制当前git分支

3、**菜单栏**还提供了不同分类的快捷功能，如果忘记快捷键，也可以看菜单里提示的快捷键。

## 3. AI agent特性

使用其自带的AI Agent特性，需要注册并登录账号；启用后在终端`shift+enter`进入`agent模式`，可以自然语言交互。

表现和手动启用`Claude Code`等CLI差不多。点击`Rich Input`方式，跟上面体验差不多，可以像终端一样`ctrl+r`查找历史记录。`Rich Input`时输入`/command`形式的提示不如原生CLI体验好。

## 4. 我的配置文件

cat settings.toml：

```sh
cat settings.toml
[appearance]

[appearance.themes]
system_theme = false
theme = "fancy_dracula"

[appearance.vertical_tabs]
enabled = true
display_granularity = "tabs"
view_mode = "expanded"
primary_info = "working_directory"
show_details_on_hover = true
show_panel_in_restored_windows = false

[appearance.text]
notebook_font_size = 14.0
font_size = 13.0
enforce_minimum_contrast = "only_named_colors"

[appearance.input]
input_mode = "waterfall"

[appearance.icon]
app_icon = "default"

[appearance.panes]
should_dim_inactive_panes = false
focus_pane_on_hover = false

[appearance.cursor]
cursor_display_type = "bar"

[general]
default_session_mode = "agent"

[agents]

[agents.third_party]
should_render_cli_agent_toolbar = true

[agents.third_party.cli_agent_toolbar_enabled_commands]
cbc = "Claude"
codebuddy = "Claude"

[agents.third_party.cli_agent_toolbar_chip_selection_setting]

[agents.third_party.cli_agent_toolbar_chip_selection_setting.custom]
left = ["file_attach", "voice_input", { context_chip = "git_diff_stats" }, "file_explorer", "rich_input"]
right = [{ context_chip = "working_directory" }, { context_chip = "shell_git_branch" }, "settings"]

[agents.warp_agent]
is_any_ai_enabled = true

[agents.warp_agent.other]
show_agent_notifications = true
show_conversation_history = true

[agents.warp_agent.input]
ai_auto_detection_enabled = true

[code]

[code.editor]
show_code_review_button = true
show_project_explorer = true
show_global_search = true
open_file_layout = "split_pane"
prefer_tabbed_editor_view = true
auto_open_code_review_pane_on_first_agent_change = true

[warp_drive]
enabled = true

[notifications]
toast_duration_secs = 8

[terminal]
maximum_grid_size = 5000

[terminal.input]
input_box_type_setting = "universal"
honor_ps1 = false

[keys]
ctrl_tab_behavior_setting = "cycle_most_recent_session"

[privacy]
telemetry_enabled = false

[privacy.secret_redaction]
enabled = false
```

## 5. 我的keybindings.yaml

cat keybindings.yaml：

```sh
---
"editor_view:insert_autosuggestion": tab
"input:open_completion_suggestions": ctrl-space
```