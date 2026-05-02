# windterm

一款很好用的SSH开源工具，基于C开发，仓库主页：[WindTerm](https://github.com/kingToolbox/WindTerm)

配置修改可参考：[WindTerm 使用教程](https://www.xiaoqiuyinboke.cn/archives/1744.html)

## 快捷键

Alt+N 新增会话
Alt+[ 或者 Alt+]，快捷切换窗口
Alt+R 显示历史
 输入字符也会自动提示历史，设置了输入2个才开始提示
Ctrl+Shift+w 关闭标签
Alt+W, Alt+M 隐藏/显示菜单栏

### 自定义快捷键

配置文件：WindTerm目录下，`global\wind.keymaps`

快捷键      Action                           功能
Ctrl+Alt+/  Window.SplitViewVertically      垂直分割（上下分屏）
Ctrl+Alt+\  Window.SplitViewHorizontally    水平分割（左右分屏）
Ctrl+Alt+K  Window.ActivatePreviousView     切换到上一个视图
Ctrl+Alt+J  Window.ActivateNextView         切换到下一个视图
Ctrl+Alt+H  script                          切换到上一个视图
Ctrl+Alt+L  script                          切换到下一个视图
Ctrl+W      Window.CloseActiveView          关闭当前视图
Ctrl+Shift+T - 复制ssh通道
Ctrl+Alt+T - 也是复制ssh通道

```sh
  { "keys": "<Ctrl+Alt+/>",
    "action": "Window.SplitViewVertically"
  },
  { "keys": "<Ctrl+Alt+\\>",
    "action": "Window.SplitViewHorizontally"
  },
  { "keys": "<Ctrl+Alt+K>",
    "action": "Window.ActivatePreviousView"
  },
  { "keys": "<Ctrl+Alt+J>",
    "action": "Window.ActivateNextView"
  },
  { "keys": "<Ctrl+Alt+H>",
    "script": "() => { window.activeKit.activatePreviousView() }"
  },
  { "keys": "<Ctrl+Alt+L>",
    "script": "() => { window.activeKit.activateNextView() }"
  },
  { "keys": "<Ctrl+W>",
    "action": "Window.CloseActiveView"
  },
    { "keys": "<Ctrl+Alt+T>",
    "when": "window.isScope('terminal.*')",
    "action": "Terminal.DuplicateSSHChannel"
  }
```

## 简化使用

* 终端左侧边缘位置，右键可以隐藏日期、行号
 也可以用快捷键打开关闭：
  `Alt+T Alt+T` 显示/关闭左侧时间列（自己关闭了）
  `Alt+T, Alt+F`  -->   折叠标记（开启了）
  `Alt+T, Alt+N` -->   行号（~~关闭了~~）
  `Alt+T, Alt+B` -->    行号代码之间是否有空格（左侧多一行，自己关闭了）
  `Alt+T, Alt+S` -->    符号（自己关闭了）

* 隐藏地址栏
 terminal/configs/toolbar.config 中  { "ToolBar" : "Standard" }  // <-----将此行移除可以去掉地址栏

* 去掉悬停标签时自动切换到这个标签
 全局设置 -> 标签 -> 悬停后激活标签页：绝不

* 去掉鼠标追踪的滚轮事件，要不vim打开文件时，滚轮会在文件中移动，不大需要
 全局设置 -> 终端 -> 鼠标追踪 -> 滚轮事件取消勾选

* 自动选中内容
 全局设置 -> 文本 -> 自动复制选定内容
* 右键粘贴内容
 全局设置 -> 终端 -> 鼠标操作，右键单机中选粘贴文本

* 删除主密码，要不每次打开windterm会输入校验
 首选项中对应的配置文件路径，到.wind\profiles\default.v10\user.config 里面，"application.masterPassword" 设置为false
* 锁屏超时，默认30分钟，可关闭
 全局设置 -> 安全中的锁屏超时，设置0即表示禁用

## 快速命令

左下角的"设置"图标（就绪上面）

## 会话日志功能

和其他终端一样，比如SecureCRT

会话->日志->开始
 而后开始操作
会话->日志->停止