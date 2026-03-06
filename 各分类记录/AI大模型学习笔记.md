# AI大模型学习

## ollama搭建个人知识库

网上资料很多，找了一篇参考：
[利用AI解读本地TXT、WORD、PDF文档](https://www.bilibili.com/read/cv33858702/)

### 安装ollama

1、从[ollama官网](https://ollama.com/download/linux)下载ollama：

`curl -fsSL https://ollama.com/install.sh | sh`

除了ollama，里面还会自动判断GPU类型并安装依赖：

```sh
[root@xdlinux ➜ /root ]$ curl -fsSL https://ollama.com/install.sh | sh
>>> Downloading ollama...
######################################################################## 100.0%#=#=-#  #       ######################################################################## 100.0%
>>> Installing ollama to /usr/local/bin...
>>> Creating ollama user...
>>> Adding ollama user to render group...
>>> Adding ollama user to video group...
>>> Adding current user to ollama group...
>>> Creating ollama systemd service...
>>> Enabling and starting ollama service...
Created symlink /etc/systemd/system/default.target.wants/ollama.service → /etc/systemd/system/ollama.service.
>>> Downloading AMD GPU dependencies...
######################################################################## 100.0%##O=#  #        ######################################################################## 100.0%
>>> The Ollama API is now available at 127.0.0.1:11434.
>>> Install complete. Run "ollama" from the command line.
>>> AMD GPU ready.
```

安装后自动起了服务：

```sh
[root@xdlinux ➜ /root ]$ netstat -anp|grep 11434
tcp        0      0 127.0.0.1:11434         0.0.0.0:*               LISTEN      23636/ollama        
[root@xdlinux ➜ /root ]$ 
```

配置局域网内访问：

.zshrc里添加，`export OLLAMA_HOST=0.0.0.0:11434`，然后重启服务，貌似没用

```sh
[root@xdlinux ➜ /root ]$ service ollama restart
Redirecting to /bin/systemctl restart ollama.service
[root@xdlinux ➜ /root ]$ netstat -anp|grep 11434
tcp        0      0 127.0.0.1:11434         0.0.0.0:*               LISTEN      23878/ollama        
[root@xdlinux ➜ /root ]$
```

在unit文件`/etc/systemd/system/ollama.service`里设置环境变量，每个环境变量各自一行

```sh
[root@xdlinux ➜ /root ]$ systemctl cat ollama.service 
# /etc/systemd/system/ollama.service
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
ExecStart=/usr/local/bin/ollama serve
User=ollama
Group=ollama
...

# 在/etc/systemd/system/ollama.service
[root@xdlinux ➜ /root ]$ vim /etc/systemd/system/ollama.service 
[root@xdlinux ➜ /root ]$ cat /etc/systemd/system/ollama.service
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
ExecStart=/usr/local/bin/ollama serve
User=ollama
Group=ollama
Restart=always
RestartSec=3
Environment="PATH=/root/.autojump/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin"
Environment="OLLAMA_HOST=0.0.0.0:11434"

[Install]
WantedBy=default.target
[root@xdlinux ➜ /root ]$
```

重新加载systemd配置并重启服务

```sh
[root@xdlinux ➜ /root ]$ systemctl daemon-reload
[root@xdlinux ➜ /root ]$ systemctl restart ollama.service
[root@xdlinux ➜ /root ]$ netstat -anp|grep 11434
tcp6       0      0 :::11434                :::*                    LISTEN      24573/ollama        
```

好了，浏览器输入：`http://192.168.1.150:11434/`，可以访问了。提示“Ollama is running"

### 下载模型

到Ollama官网，点击右上角的Models即进入：[ollama library](https://ollama.com/library)

暂时使用阿里的通义千问：qwen2 1.5b 实验

复制提示的命令，此处为`ollama run qwen2:1.5b`，到linux终端运行，其会自动先`ollama pull qwen2:1.5b`再`run`


## Claude Code

### 安装

[Claude Code 安装与使用](https://www.runoob.com/claude-code/claude-code-install.html)

安装方式：
1、方式1：官网脚本 `curl -fsSL https://claude.ai/install.sh | bash`
* 可能会被墙导致下载不到脚本，报错：`bash: line 1: syntax error near unexpected token <'`

2、方式2：使用npm安装（推荐）
* 先安装Node.js，Node.js 18 或更新版本环境（OpenClaw需要v22.0 或更高版本，还是尽量安装新一些的版本）
    * node.js官网下载二进制包 -> 解压 -> 添加PATH环境变量。
    * `node -v`检查，如`v24.13.0`
    * `npm -v`检查，如`11.6.2`
    * 设置npm国内镜像：`npm config set registry https://registry.npmmirror.com/`

* npm安装claude code： `npm install -g @anthropic-ai/claude-code`

```sh
[root@xdlinux ➜ /home ]$ npm install -g @anthropic-ai/claude-code
changed 3 packages in 591ms
2 packages are looking for funding
  run `npm fund` for details
```

安装后查看`claude`是否安装正常

```sh
[root@xdlinux ➜ /home ]$ claude --version
2.1.63 (Claude Code)
```

### 登录Claude Code（默认连接官网）

第一次启动 Claude Code 时,需要登录账号：执行`claude`，会默认连`Anthropic`官方API，但是国内网络不通。
* 不使用`Claude`自家模型的话，可以通过`Anthropic Messages`协议（`LLM Gateway`模式+`base_url`），指定第三方模型，比如千问、deepseek等
    * 配置方式可参考：[Claude Code API 配置](https://www.runoob.com/claude-code/claude-code-setup.html)

下面是默认情况下连接`Anthropic`官方报错：

```sh
[root@xdlinux ➜ /home ]$ claude
Welcome to Claude Code v2.1.63
…………………………………………………………………………………………………………………………………………………………

     *                                       █████▓▓░
                                 *         ███▓░     ░░
            ░░░░░░                        ███▓░
    ░░░   ░░░░░░░░░░                      ███▓░
   ░░░░░░░░░░░░░░░░░░░    *                ██▓░░      ▓
                                             ░▓▓███▓▓░
 *                                 ░░░░
                                 ░░░░░░░░
                               ░░░░░░░░░░░░░░░░
       █████████                                        *
      ██▄█████▄██                        *
       █████████      *
…………………█ █   █ █………………………………………………………………………………………………………………

 Unable to connect to Anthropic services

 Failed to connect to api.anthropic.com: ERR_BAD_REQUEST

 Please check your internet connection and network settings.

 Note: Claude Code might not be available in your country. Check supported countries at https://anthropic.com/supported-countries
[<u]9;4;0;#                    
```

### Claude Code API 配置

可见：[Claude Code API 配置](https://www.runoob.com/claude-code/claude-code-setup.html)

以阿里百炼为例，比如我使用的是`Coding Plan`套餐
* 需要配置的环境变量：
    * `ANTHROPIC_BASE_URL`，配置为：`https://coding.dashscope.aliyuncs.com/apps/anthropic`
    * `ANTHROPIC_AUTH_TOKEN`，配置为所购买套餐对应的API Key
    * `ANTHROPIC_MODEL`，设置为 Coding Plan [支持的模型](https://help.aliyun.com/zh/model-studio/coding-plan?spm=0.0.0.i5)。
        * 推荐模型：qwen3.5-plus（支持图片理解）、kimi-k2.5（支持图片理解）、glm-5、MiniMax-M2.5
        * 更多模型：qwen3-max-2026-01-23、qwen3-coder-next、qwen3-coder-plus、glm-4.7
* 详情可见百炼官网上API接入各工具的说明：[Coding Plan -- 接入AI工具](https://bailian.console.aliyun.com/cn-beijing/?tab=doc#/doc/?type=model&url=3023078)

1、在`claude`的配置文件里进行持久化设置，没有则新建`~/.claude/settings.json`：

```sh
{
    "env": {
        "ANTHROPIC_AUTH_TOKEN": "sk-sp-10faxxxxxxxxxxxx",
        "ANTHROPIC_BASE_URL": "https://coding.dashscope.aliyuncs.com/apps/anthropic",
        "ANTHROPIC_MODEL": "qwen3.5-plus"
    }
}
```

2、编辑或新增 `~/.claude.json` 文件，将`hasCompletedOnboarding` 字段的值设置为`true`并保存文件

该步骤可避免启动Claude Code时报错：`Unable to connect to Anthropic services`

```sh
{
  "hasCompletedOnboarding": true
}
```

### 运行

cd到项目目录，启动`claude`，即可通过终端方式进行交互。

```sh
╭─── Claude Code v2.1.63 ──────────────────────────────────────────────────────────────────────────────────╮
│                                      │ Tips for getting started                                          │
│             Welcome back!            │ Run /init to create a CLAUDE.md file with instructions for Claude │
│                                      │ ───────────────────────────────────────────────────────────────── │
│                ▐▛███▜▌               │ Recent activity                                                   │
│               ▝▜█████▛▘              │ No recent activity                                                │
│                 ▘▘ ▝▝                │                                                                   │
│                                      │                                                                   │
│   qwen3.5-plus · API Usage Billing   │                                                                   │
│                /home                 │                                                                   │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────╯

  /model to try Opus 4.6

──────────────────────────────────────────────────────────────────────────────────────────
❯  
```

### 常见命令

也可见阿里云上的[常见命令](https://bailian.console.aliyun.com/cn-beijing/?tab=doc#/doc/?type=model&url=3023078)

* `/init`：在项目根目录生成 CLAUDE.md 文件，用于定义项目级指令和上下文。
* `/status`：查看当前模型、API Key、Base URL 等配置状态
* `/clear`：清除对话历史，开始全新对话
* `/plan`：进入规划模式，仅分析和讨论方案，不修改代码
* `/compact`：压缩对话历史，释放上下文窗口空间
* `/config`：打开配置菜单，可设置语言、主题等

### 使用经验

安装后，就可以进行对话了，但是默认的权限是比较受限的，自己的环境可以按需放开一些权限：命令执行、编辑等。

## OpenClaw

### 安装

参考：
* [阿里云百炼 -- 接入OpenClaw](https://bailian.console.aliyun.com/cn-beijing/?tab=doc#/doc/?type=model&url=3023085)
* [OpenClaw (Clawdbot) 教程](https://www.runoob.com/ai-agent/openclaw-clawdbot-tutorial.html)

方式1：`curl -fsSL https://openclaw.ai/install.sh | bash`
方式2：`npm install -g openclaw@latest`

```sh
[root@xdlinux ➜ ~ ]$ openclaw -v
2026.3.2

[root@xdlinux ➜ ~ ]$ npm list -g openclaw
/home/workspace/local/node-v24.13.0-linux-x64/lib
└── openclaw@2026.3.2
```

### 初始化OpenClaw并配置API

1、初始化并安装后台服务：`openclaw onboard --install-daemon`

按照提示选择，可见：[阿里云百炼 -- 接入OpenClaw](https://bailian.console.aliyun.com/cn-beijing/?tab=doc#/doc/?type=model&url=3023085)

2、配置模型API，此处用的百炼Coding Plan

1）启动web页面：`openclaw dashboard`

```sh
[root@xdlinux ➜ ~ ]$ openclaw dashboard

🦞 OpenClaw 2026.3.2 (85377a2) — Your config is valid, your assumptions are not.

Dashboard URL: http://127.0.0.1:18789/#token=1d0acfe9d056923f0539098763ae5a2a3c0ca6a719c4fdb8
Copy to clipboard unavailable.
No GUI detected. Open from your computer:
ssh -N -L 18789:127.0.0.1:18789 root@192.168.1.150
Then open:
http://localhost:18789/
http://localhost:18789/#token=1d0acfe9d056923f0539098763ae5a2a3c0ca6a719c4fdb8
Docs:
https://docs.openclaw.ai/gateway/remote
https://docs.openclaw.ai/web/control-ui
```

2）并在笔记本上（MacOS）创建本地端口转发（SSH 隧道）：`ssh -N -L 18789:127.0.0.1:18789 root@192.168.1.150`
* `-N`：不执行远程命令，只建立隧道（不分配终端）
* `-L`：本地端口转发参数
* `18789:127.0.0.1:18789`端口转发规则

```sh
# 数据流向：
# 当你的本地程序访问 localhost:18789，SSH 会通过加密隧道，将数据转发到远程服务器的 127.0.0.1:18789
# 远程服务器上的服务（如 OpenClaw）处理请求，响应数据通过同样的加密隧道返回
你的电脑 (本地)               远程服务器 (192.168.1.150)
    |                               |
本地端口:18789 <---- SSH隧道 ----> 远程端口:18789
    |                               |
    |                            (运行着服务)
你的应用                         localhost:18789
(浏览器/客户端)                   (如 OpenClaw)
```

3）笔记本就可连接了：`http://localhost:18789/`

试过`http://192.168.1.150:18789/`方式访问，加了白名单还是不通，所以还是用上面的SSH隧道方式
```sh
[root@xdlinux ➜ ~ ]$ firewall-cmd --permanent --add-port=18789/tcp
[root@xdlinux ➜ ~ ]$ firewall-cmd --permanent --add-port=18789/udp
[root@xdlinux ➜ ~ ]$ firewall-cmd --reload
```

更新：原因是gateway监听模式是`loopback`，只监听了localhost

```sh
"gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "loopback",
    "auth": {
      "mode": "token",
      "token": "__OPENCLAW_REDACTED__"
    },
    ...
```
调整`bind`为`lan`模式，并设置校验token（比如admin@1234），然后重启gateway服务：`openclaw gateway restart`
```sh
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "lan",
    "controlUi": {
      "enabled": true,
      "allowInsecureAuth": true
    },
    "auth": { 
      "mode": "token",
      "token": "admin@1234"
    }
    ...
```
貌似没用，需要curl手动指定头`Authorization: Bearer`，浏览器没办法指定

有问题可以使用 doctor 命令修复配置并重启：`openclaw doctor --fix` （善用doctor）


4）按上面百炼的说明步骤：修改Raw JSON配置，注意找到对应的Json Key替换。然后点击`Save` -> `Update`。

然后就可以开始聊天了。可以在上面的网页链接；也可以在终端。

### 接入飞书

电报、Slack可能要梯子，先用飞书。
网上教程很多，可参考：[保姆级教程】手把手教你安装OpenClaw并接入飞书](https://cloud.tencent.com/developer/article/2626160)

1、到飞书上创建机器人并发布应用
2、OpenClaw安装飞书插件：`openclaw plugins install @m1heng-clawd/feishu`

### 设置权限

`openclaw 2026.3.2`的版本中，OpenClaw 默认的权限策略发生了变化：默认情况下，Agent只允许进行纯对话。出于安全考量，3.2版本将工具执行权限与聊天能力彻底隔离。

**Profile（配置/档案）**：OpenClaw的Profile是一个用来管理和切换不同身份、权限、运行环境或行为模式的核心概念。它允许你为不同的使用场景创建独立的“配置快照”，从而在多任务、多身份、多权限级别之间灵活切换，同时保证隔离性和安全性。
  Profile让你可以“精养”多只不同性格、不同能力、不同权限的“龙虾”，让它们各司其职，共同为你服务，同时避免互相干扰和越权操作。

可以进行调整，可参考：[OpenClaw 2026.3.2 版本权限隔离导致工具失效，两招教你满血复活！](https://www.xmsumi.com/detail/2625)

步骤：
* 查看当前的`Profile`状态，`openclaw config get tools`

```sh
[root@xdlinux ➜ tmpdir ]$ openclaw config get tools
🦞 OpenClaw 2026.3.2 (85377a2) — Your second brain, except this one actually remembers where you left things.
{
  "profile": "messaging"
}
```

修改`~/.openclaw/openclaw.json`：

```sh
"tools": {
    "profile": "full"
  },
```

```sh
openclaw config get tools
```


### 安装Skills（技能）

1、先安装技能管理工具`clawhub`，这是安装和管理技能的“应用商店”。  
安装：`npm install -g clawhub`

查看版本：
```sh
[root@xdlinux ➜ .openclaw ]$ clawhub -V
0.7.0
```

2、

## 文档学习

[awesome-openclaw-usecases](https://github.com/hesamsheikh/awesome-openclaw-usecases)
    https://github.com/cogine-ai/awesome-openclaw-zh/tree/main
[awesome-openclaw-skills](https://github.com/VoltAgent/awesome-openclaw-skills)
[微软：ai-agents-for-beginners](https://github.com/microsoft/ai-agents-for-beginners)

