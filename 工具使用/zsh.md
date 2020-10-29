## zsh说明

介绍说明参考，若要安装，使用下一章节的步骤：

[MacTalk 终极 Shell](http://macshuo.com/?p=676)

1. git插件

    在git受控的目录下，会显示git相关信息，另外对git命令做了很多简化。例如 gco=’git checkout’、gd=’git diff’、gst=’git status’、g=’git’等等

2. autojump插件

    十分好用的跳转功能。

    直接输入目录名跳转

    ..直接回到上层

    j 部分目录名直接跳转

* 各种补全：

    路径补全、命令补全，命令参数补全，插件内容补全等等。

        触发补全只需要按一下或两下 tab 键，

        补全项可以使用 ctrl+n/p/f/b 下/上/右/左切换。 **方向切换**

        比如你想杀掉 java 的进程，只需要输入 kill java + tab键，如果只有一个 java 进程，zsh 会**自动替换为进程的 pid**，如果有多个则会出现选择项供你选择。

        ssh + 空格 + 两个tab键，zsh会列出所有访问过的主机和用户名进行补全

* 智能跳转

    安装了autojump之后，zsh 会自动记录你访问过的目录，

    通过 j + 目录名 可以直接进行目录跳转，而且目录名支持模糊匹配和自动补全(比如，/home/xd/workspace，如果之前访问过，j wo 回车就会cd进入该路径)

* 目录浏览和跳转：

    输入 d，即可列出你在这个会话里访问的目录列表，输入列表前的序号，即可直接跳转。 (很方便的功能)

* 在当前目录下输入 .. 或 … (cd主目录) ，或**直接输入当前目录名**都可以跳转，你甚至不再需要输入 cd 命令了。(功能同: ..相当于alias了一个跳到前一文件夹的的别名)


## centos下使用

一开始通过root下，使用yum install zsh下载后配置有问题， yum remove zsh卸载后，登录用户输入密码一直登录不了。

卸载时可能有什么依赖项被删除了。重新root下install后还原。还是建议通过官网步骤进行操作。

### 通过官网步骤安装：

https://github.com/robbyrussell/oh-my-zsh/

via curl：

```sh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

* 发现链接连不上了，手动复制脚本内容到新增文件install.sh，然后`sh -c ./install.sh`
    - [install.sh](https://github.com/ohmyzsh/ohmyzsh/blob/master/tools/install.sh)

* 若提示要安装zsh，则`yum install zsh` (CentOS)
* zsh中`history`显示命令历史的时间，可用`history -i`，不过vi打开文件看不到时间，可配合`less`在终端查看

### PS提示显示

把 c 改为 d，c 表示当前目录，d 表示绝对路径，另外在末尾增加了一个「 > 」

在.zshrc中添加

```sh
# 把 c 改为 d，c 表示当前目录，d 表示绝对路径，显示格式"[➜ /home/xd ]$"
PROMPT='[%{$fg_bold[red]%}%n@%M ➜ %{$fg_bold[green]%}%p%{$fg[cyan]%}%d %{$fg_bold[blue]%}$(git_prompt_info)%{$fg_bold[blue]%}% %{$reset_color%}]$ '
# 把 c 改为 d，c 表示当前目录，d 表示绝对路径，另外在末尾增加了一个「 > 」
#PROMPT='%{$fg_bold[red]%}➜ %{$fg_bold[green]%}%p%{$fg[cyan]%}%d %{$fg_bold[blue]%}$(git_prompt_info)%{$fg_bold[blue]%}% %{$reset_color%}>'
#PROMPT='%{$fg_bold[red]%}➜ %{$fg_bold[green]%}%p %{$fg[cyan]%}%c %{$fg_bold[blue]%}$(git_prompt_info)%{$fg_bold[blue]%} % %{$reset_color%}'
```

* 参考设置
    - [oh-my-zsh终端用户名设置（PS1）](https://www.jianshu.com/p/bf488bf22cba)
    - 为了显示用户名，把 %n 加到➜前面

```
code    info
%T  系统时间（时：分）
%*  系统时间（时：分：秒）
%D  系统日期（年-月-日）
%n  你的用户名
%B - %b 开始到结束使用粗体打印
%U - %u 开始到结束使用下划线打印
%d  你目前的工作目录
%~  你目前的工作目录相对于～的相对路径
%M  计算机的主机名
%m  计算机的主机名（在第一个句号之前截断）
%l  你当前的tty
%n  登录名
```

### 主题

默认主题是：ZSH_THEME=”robbyrussell”, .zshrc中有该配置项

oh my zsh 提供了数十种主题，相关文件在~/.oh-my-zsh/themes目录下

### 插件

oh my zsh 项目提供了完善的插件体系，相关的文件在~/.oh-my-zsh/plugins目录下

插件也是在.zshrc里配置，找到plugins关键字，你就可以加载自己的插件了，系统默认加载 git ，你可以在后面追加内容

#### 使用autojump

智能跳转，安装了autojump之后，zsh 会自动记录你访问过的目录，通过 j + 目录名 可以直接进行目录跳转，而且目录名支持模糊匹配和自动补全，例如你访问过hadoop-1.0.0目录，输入j hado 即可正确跳转。j –stat 可以看你的历史路径库。

1. 先下载安装autojump

```sh
    如果是 Linux，去下载 autojump 的最新版本，比如：

    git clone git://github.com/joelthelion/autojump.git
    解压缩后进入目录，执行

    ./install.py
    最后把以下代码加入.zshrc：

    [[ -s ~/.autojump/etc/profile.d/autojump.sh ]] && . ~/.autojump/etc/profile.d/autojump.sh
```

2. .zshrc中追加：

```sh
plugins=(git autojump)

[[ -s ~/.autojump/etc/profile.d/autojump.sh ]] && . ~/.autojump/etc/profile.d/autojump.sh
```

### 配置文件备份

```sh
# autojump
[[ -s ~/.autojump/etc/profile.d/autojump.sh ]] && . ~/.autojump/etc/profile.d/autojump.sh
# 提示
PROMPT='[%{$fg_bold[red]%}%n@%M ➜ %{$fg_bold[green]%}%p%{$fg[cyan]%}%d %{$fg_bold[blue]%}$(git_prompt_info)%{$fg_bold[blue]%}% %{$reset_color%}]$ '

# 可选 PROMPT="[%F{135}%n%f@%F{166}%m%f %F{118}% %d%f]$ "

alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
alias vi='vim'
alias la='ls -a'
alias ..='cd ..'
# man 彩色
export LESS_TERMCAP_mb=$'\E[01;31m'
export LESS_TERMCAP_md=$'\E[01;31m'
export LESS_TERMCAP_me=$'\E[0m'
export LESS_TERMCAP_se=$'\E[0m'
export LESS_TERMCAP_so=$'\E[01;44;33m'
export LESS_TERMCAP_ue=$'\E[0m'

# 不要设置PS1，会与PROMPT冲突
# PS1="\[\e[32m\][\u@\h \w]$\[\e[m\] "
export HISTTIMEFORMAT="%F %T `whoami` "
```

## 卸载方法

https://github.com/robbyrussell/oh-my-zsh/

执行`uninstall_oh_my_zsh`

If you want to uninstall oh-my-zsh, just run uninstall_oh_my_zsh from the command-line. It will remove itself and revert your previous bash or zsh configuration.