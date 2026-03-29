# WezTerm 默认快捷键大全

以下是WezTerm官方默认快捷键，按功能分类整理，同时标注Windows/Linux与macOS系统差异。可通过`wezterm show-keys --lua`命令查看完整配置，或在`~/.wezterm.lua`中自定义修改。

---

### 一、基础操作

| 快捷键 | 功能 | 系统 |
|-------|-----|-----|
| **Ctrl+Shift+C** / **Cmd+C** | 复制到剪贴板 | 全平台 |
| **Ctrl+Shift+V** / **Cmd+V** | 从剪贴板粘贴 | 全平台 |
| **Ctrl+Insert** | 复制到主选择区 | 全平台 |
| **Shift+Insert** | 从主选择区粘贴 | 全平台 |
| **Ctrl+Shift+P** / **Cmd+Shift+P** | 打开命令面板 | 全平台 |
| **Ctrl+Shift+F** / **Cmd+F** | 搜索滚动历史 | 全平台 |
| **Ctrl+Shift+X** | 进入复制模式 | 全平台 |
| **Ctrl+Shift+Z** | 撤销 | 全平台 |
| **Ctrl+Shift+Y** | 重做 | 全平台 |
| **Ctrl+Shift+Backspace** | 清除当前窗格历史 | 全平台 |

---

### 二、标签页管理

| 快捷键 | 功能 | 系统 |
|-------|-----|-----|
| **Ctrl+Shift+T** / **Cmd+T** | 新建标签页 | 全平台 |
| **Ctrl+Shift+W** / **Cmd+W** | 关闭当前标签页 | 全平台 |
| **Ctrl+PageUp** / **Cmd+Option+Left** | 切换到上一个标签页 | 全平台 |
| **Ctrl+PageDown** / **Cmd+Option+Right** | 切换到下一个标签页 | 全平台 |
| **Ctrl+Shift+PageUp** / **Cmd+Shift+Left** | 向左移动标签页 | 全平台 |
| **Ctrl+Shift+PageDown** / **Cmd+Shift+Right** | 向右移动标签页 | 全平台 |
| **Alt+数字键** / **Cmd+数字键** | 切换到指定序号标签页(1-9) | 全平台 |

---

### 三、窗格(分屏)操作

| 快捷键 | 功能 | 系统 |
|-------|-----|-----|
| **Ctrl+Shift+\|** | 垂直分屏(创建新窗格) | 全平台 |
| **Ctrl+Shift+-** | 水平分屏(创建新窗格) | 全平台 |
| **Ctrl+Shift+方向键** | 在窗格间切换 | 全平台 |
| **Ctrl+Shift+Space** | 激活窗格选择模式 | 全平台 |
| **Alt+方向键** | 调整窗格大小(每次1单元格) | 全平台 |
| **Alt+Shift+方向键** | 快速调整窗格大小(每次5单元格) | 全平台 |
| **Ctrl+Shift+Q** | 关闭当前窗格 | 全平台 |
| **Ctrl+Shift+Z** | 切换窗格全屏模式 | 全平台 |

---

### 四、窗口管理

| 快捷键 | 功能 | 系统 |
|-------|-----|-----|
| **Ctrl+Shift+N** / **Cmd+N** | 新建窗口 | 全平台 |
| **Alt+Enter** | 切换全屏 | 全平台 |
| **Ctrl+Shift+Enter** | 在新窗口中打开当前进程 | 全平台 |
| **Ctrl+Shift+M** | 最大化窗口 | 全平台 |
| **Ctrl+Shift+F11** | 切换到无边界模式 | 全平台 |

---

### 五、字体与显示

| 快捷键 | 功能 | 系统 |
|-------|-----|-----|
| **Ctrl+Plus** / **Cmd+Plus** | 增大字体大小 | 全平台 |
| **Ctrl+Minus** / **Cmd+Minus** | 减小字体大小 | 全平台 |
| **Ctrl+0** / **Cmd+0** | 重置字体大小 | 全平台 |
| **Ctrl+Shift+L** | 重新加载配置 | 全平台 |
| **Ctrl+Shift+R** | 重新连接 | 全平台 |

---

### 六、复制模式快捷键(进入后)

| 快捷键 | 功能 |
|-------|-----|
| **Escape** | 退出复制模式 |
| **Ctrl+A** | 移动到行首 |
| **Ctrl+E** | 移动到行尾 |
| **Ctrl+F** | 向前移动一个字符 |
| **Ctrl+B** | 向后移动一个字符 |
| **Ctrl+P** | 向上移动一行 |
| **Ctrl+N** | 向下移动一行 |
| **Ctrl+V** | 粘贴 |
| **Ctrl+K** | 从光标处删除到行尾 |
| **Ctrl+U** | 从光标处删除到行首 |

---

### 七、其他常用功能

| 快捷键 | 功能 | 系统 |
|-------|-----|-----|
| **Ctrl+Shift+S** | 保存当前窗格内容 | 全平台 |
| **Ctrl+Shift+L** | 重新加载配置 | 全平台 |
| **Ctrl+Shift+D** | 显示调试信息 | 全平台 |
| **Ctrl+Shift+I** | 打开开发者工具 | 全平台 |

---

### 八、自定义快捷键示例

在`~/.wezterm.lua`中添加以下配置可修改默认快捷键：

```lua
config.keys = {
  -- 禁用默认的Ctrl+Shift+W关闭标签页
  {
    key = 'W',
    mods = 'CTRL|SHIFT',
    action = wezterm.action.DisableDefaultAssignment,
  },
  -- 自定义Alt+W关闭标签页
  {
    key = 'W',
    mods = 'ALT',
    action = wezterm.action.CloseCurrentTab { confirm = true },
  },
}
```

---

### 九、实用命令

1. 查看当前快捷键配置：`wezterm show-keys --lua`
2. 查看默认配置文档：访问https://wezfurlong.org/wezterm/config/default-keys.html
3. 配置文件位置：`~/.wezterm.lua`(Linux/macOS)或`%USERPROFILE%\.wezterm.lua`(Windows)

需要我按使用场景（如日常终端、分屏工作流、远程开发）整理一份精简的高频快捷键清单吗？