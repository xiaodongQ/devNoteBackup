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

