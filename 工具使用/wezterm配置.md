# WezTerm 使用和快捷键

配置可以参考：[wezterm-config](https://github.com/QianSong1/wezterm-config)，使用了一下，快捷键和样式都挺好，也[fork](https://github.com/xiaodongQ/wezterm-config)了一份。

进一步提升体验：安装`PowerShell 7`，并且`wezterm`脚本里配置，可体验新版本PowerShell带来的效率提升（如自动补全）
- 比如这里的配置：[wezterm配置分享](https://zhuanlan.zhihu.com/p/1973514411802108537)

# QianSong1/wezterm-config 配置速查（快捷键+技巧）
本文基于 https://github.com/QianSong1/wezterm-config 整理，仅提取仓库内原生快捷键与配置技巧。

---

## 一、前置要求
1. 安装 WezTerm 终端
   - Windows 示例路径：`C:\soft\WezTerm-windows`
2. 安装 Nerd Font 字体
   - 推荐：`MesloLGM Nerd Font`、`JetBrainsMono NF`
   - 版本：**v3.2.1**（版本必须与图标匹配，否则乱码）

---

## 二、配置部署方法
1. 下载仓库压缩包并解压
2. 放入配置目录
   - 通用路径：`$HOME/.config/wezterm`
   - Windows 路径：`C:\Users\Fizz\.config\wezterm`

---

## 三、完整快捷键清单
| 操作 | 快捷键 |
| :--- | :--- |
| 复制 | Ctrl+Shift+C |
| 粘贴 | Ctrl+Shift+V |
| 重命名标签栏 | Ctrl+Shift+R |
| 水平拆分窗格（左右） | Ctrl+Alt+\ |
| 垂直拆分窗格（上下） | Ctrl+Alt+/ |
| 关闭当前窗格 | Ctrl+Alt+- |
| 最大化/最小化当前窗格 | Ctrl+Alt+Z |
| 全屏 | F11 |
| 向上扩展窗格 | Ctrl+Alt+↑ |
| 向下扩展窗格 | Ctrl+Alt+↓ |
| 向左扩展窗格 | Ctrl+Alt+← |
| 向右扩展窗格 | Ctrl+Alt+→ |
| 放大字体 | Alt+↑ |
| 缩小字体 | Alt+↓ |
| 重置字体大小 | Alt+R |

---

## 四、Windows 右键菜单配置（核心技巧）
实现「在此处打开 WezTerm」右键菜单：
1. Win+R → 输入 `regedit` 打开注册表
2. 定位到：`HKEY_CLASSES_ROOT\Directory\Background\shell`
3. 新建项 `wezterm`
   - `Icon`：指向 WezTerm 安装程序图标
   - `(默认)`：设置菜单名，如 `Open Wezterm Here`
4. 在 `wezterm` 下新建项 `command`
   - `(默认)` 赋值：
     ```
     "C:\soft\WezTerm-windows\wezterm-gui" start --no-auto-connect --cwd "%V\\"
     ```

---

## 五、关键注意事项
- 字体版本与图标必须对应，否则出现乱码
- 配置文件需放入指定目录，否则不生效
- 配置基于 Lua 编写，适配 Windows/macOS/Linux

---

## mac下的快捷键

在 macOS 下，快捷键配置如下（`mod.SUPER` = `SUPER` 即 Command 键）：

**基础功能：**
| 快捷键 | 功能 |
|--------|------|
| F1 | 进入复制模式 |
| F2 | 命令面板 |
| F3 | 启动器 |
| F4 | 标签导航 |
| F11 | 全屏切换 |
| F12 | 调试覆盖 |
| `⌘ + f` | 搜索 |
| `Ctrl + Shift + c` | 复制到剪贴板 |
| `Ctrl + Shift + v` | 从剪贴板粘贴 |

**标签页：**
| 快捷键 | 功能 |
|--------|------|
| `⌘ + t` | 新建标签页 |
| `⌘ + Ctrl + t` | 新建 WSL 标签页 |
| `⌘ + Ctrl + w` | 关闭标签页 |
| `⌘ + [` | 上一个标签 |
| `⌘ + ]` | 下一个标签 |
| `⌘ + Ctrl + [` | 左移标签 |
| `⌘ + Ctrl + ]` | 右移标签 |

**窗口：**
| 快捷键 | 功能 |
|--------|------|
| `⌘ + n` | 新建窗口 |

**分屏：**
| 快捷键 | 功能 |
|--------|------|
| `⌘ + Ctrl + /` | 垂直分割 |
| `⌘ + Ctrl + \` | 水平分割 |
| `⌘ + Ctrl + -` | 关闭分屏 |
| `⌘ + Ctrl + z` | 分屏缩放切换 |
| `⌘ + w` | 关闭当前分屏 |
| `⌘ + Ctrl + k/j/h/l` | 切换分屏方向 |
| `⌘ + Ctrl + ↑/↓/←/→` | 调整分屏大小 |

**字体：**
| 快捷键 | 功能 |
|--------|------|
| `⌘ + ↑` | 增大字体 |
| `⌘ + ↓` | 减小字体 |
| `⌘ + r` | 重置字体大小 |

**Key Tables（前缀键 `Ctrl + Shift + Space`）：**
| 快捷键 | 功能 |
|--------|------|
| `Ctrl + Shift + Space + f` → `k/j/r` | 调整字体大小 |
| `Ctrl + Shift + Space + p` → `k/j/h/l` | 调整分屏大小 |
| `Ctrl + Shift + Space + q` 或 `Escape` | 退出 |

**其他：**
| 快捷键 | 功能 |
|--------|------|
| `Ctrl + Shift + R` | 重命名标签 |
| `Ctrl + 左键点击` | 打开链接 |