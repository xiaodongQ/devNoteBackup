# docker

## previous

## 入门

[Docker 入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)

### 背景

* 背景
    - 2013年发布至今， Docker 一直广受瞩目，被认为可能会改变软件行业。
    - 环境配置的难题
        + 软件开发最大的麻烦事之一，就是环境配置。用户计算机的环境都不相同，而用户必须保证两件事：操作系统的设置，各种库和组件的安装。只有它们都正确，软件才能运行。如果某些老旧的模块与当前环境不兼容，那就麻烦了。环境配置如此麻烦，换一台机器，就要重来一次，旷日费时。很多人想到，能不能从根本上解决问题，软件可以带环境安装？也就是说，安装的时候，把原始环境一模一样地复制过来。
    - 虚拟机（virtual machine）就是带环境安装的一种解决方案。
        + 它可以在一种操作系统里面运行另一种操作系统，比如在 Windows 系统里面运行 Linux 系统。应用程序对此毫无感知
        + 虽然用户可以通过虚拟机还原软件的原始环境。但是，这个方案有几个缺点。
            * 资源占用多
                - 虚拟机会独占一部分内存和硬盘空间。它运行的时候，其他程序就不能使用这些资源了。哪怕虚拟机里面的应用程序，真正使用的内存只有 1MB，虚拟机依然需要几百 MB 的内存才能运行。
            * 冗余步骤多
                - 虚拟机是完整的操作系统，一些系统级别的操作步骤，往往无法跳过，比如用户登录。
            * 启动慢
                - 启动操作系统需要多久，启动虚拟机就需要多久。可能要等几分钟，应用程序才能真正运行。
    - Linux 容器
        + 由于虚拟机存在这些缺点，Linux 发展出了另一种虚拟化技术：Linux 容器（Linux Containers，缩写为 LXC）。
        + Linux 容器不是模拟一个完整的操作系统，而是对进程进行隔离。
        + 由于容器是进程级别的，相比虚拟机有很多优势。
            * 启动快
                - 容器里面的应用，直接就是底层系统的一个进程，而不是虚拟机内部的进程。所以，启动容器相当于启动本机的一个进程，而不是启动一个操作系统，速度就快很多。
            * 资源占用少
                - 容器只占用需要的资源，不占用那些没有用到的资源；虚拟机由于是完整的操作系统，不可避免要占用所有资源。
                - 另外，多个容器可以共享资源，虚拟机都是独享资源。
            * 体积小
                - 容器只要包含用到的组件即可，而虚拟机是整个操作系统的打包，所以容器文件比虚拟机文件要小很多。
    - 什么是Docker
        + Docker 属于 Linux 容器的一种封装，提供简单易用的容器使用接口。它是目前最流行的 Linux 容器解决方案。
        + Docker 将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。有了 Docker，就不用担心环境问题。
        + 总体来说，Docker 的接口相当简单，用户可以方便地创建和使用容器，把自己的应用放入容器。容器还可以进行版本管理、复制、分享、修改，就像管理普通的代码一样。

### Docker用途

* Docker用途
    - Docker 的主要用途，目前有三大类。
    - 1. 提供一次性的环境。
        + 比如，本地测试他人的软件、持续集成的时候提供单元测试和构建的环境。
    - 2. 提供弹性的云服务。
        + 因为 Docker 容器可以随开随关，很适合动态扩容和缩容。
    - 3. 组建微服务架构。
        + 通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构。

### Docker安装

* Docker安装
    - Docker 是一个开源的商业产品，有两个版本：社区版（Community Edition，缩写为 CE）和企业版（Enterprise Edition，缩写为 EE）。企业版包含了一些收费服务，个人开发者一般用不到。下面的介绍都**针对社区版**。
    - Ubuntu
        + [Get Docker Engine - Community for Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
        + 要求Ubuntu系统版本
            * Disco 19.04、Cosmic 18.10、Bionic 18.04 (LTS)、Xenial 16.04 (LTS)
            * 查看自己Ubuntu虚拟机的版本：
                - `lsb_release -a`，可以看到相关行：`Release:    16.04`、`Codename:   xenial`
                - 所以满足要求
        + 卸载老版本
            * 老版本docker称作docker, docker.io , 或者 docker-engine，如果安装了则卸载：`apt-get remove docker docker-engine docker.io containerd runc`
        + 安装社区版Docker引擎，使用仓库方式安装(建议。也可手动通过包安装)
            * 先设置仓库
                - 1. 设置仓库 `sudo apt-get update`
                - 2. 安装相关包以是get能用http(这步也可省略，一般都支持，若已有这些包执行也没关系)
                    + `sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common`
                - 3. 添加Docker官方GPG Key
                    + `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`
                    + GnuPG软件（简称GPG），它是目前最流行、最好用的加密工具之一。
                        * 使用GPG能轻松传递加密信息
                        * [GPG入门教程](http://www.ruanyifeng.com/blog/2013/07/gpg.html)
                - 4. 设置稳定的仓库地址
                    + `sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"`
                    + 由于ubuntu下载了最新的19.10(代号`eoan`)，在仓库`ttps://download.docker.com/linux/ubuntu`中并没有，所以写19.04的代号`disco`(注意需要小写！因为大小写试了n个代号都是没有找到Release)
                        * `sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu disco stable"`
                        * 对于已添加的`eoan`代号源，使用`--remove`来删除，`sudo add-apt-repository --remove "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"`
                        * [Ubuntu 各版本代号简介](https://blog.csdn.net/zhengmx100/article/details/78352773)
            * 开始安装(不设置仓库进行安装会找不到docker的包)
                - `sudo apt-get update`
                - `sudo apt-get install docker-ce docker-ce-cli containerd.io`
                    + 执行由于部分库版本不满足，需要升级ubuntu，虚拟机里一直升级失败，所以用指定版本的方式升级
                -  查看当前ubuntu版本能安装的版本(**下面由于版本问题安装后冲突或者依赖不满足，并不能正常运行使用。还是安装一个新的满足Docker版本要求的Ubuntu版本重新安装Docker**)
                    + 查看 docker-ce 满足版本 `apt-cache madison docker-ce`
                        * 尝试列表里很多新一点的版本都有部分库版本不满足要求，最终选择了：17.09.1~ce-0~ubuntu
                        * 使用 `sudo apt-get install docker-ce=17.09.1~ce-0~ubuntu`
                    + 查看 docker-ce-cli 满足版本 `apt-cache madison docker-ce-cli`
                        * `sudo apt-get install docker-ce-cli=5:18.09.0~3-0~ubuntu-xenial` 安装时会卸载docker-ce。。。暂时不安装
                    + 查看 containerd.io 满足版本`apt-cache madison containerd.io`
                        * `sudo apt-get install containerd.io=1.2.0~beta.2-1`，也要apt-cache madison查看能安装的版本。。。
    - CentOS
        + [Get Docker Engine - Community for CentOS](https://docs.docker.com/install/linux/docker-ce/centos/)
        + 步骤参考链接：
            * 卸载老版本 `sudo yum remove docker docker-client docker-client-latest docker-common docker-latestdocker-latest-logrotate docker-logrotate docker-engine`
            * 安装社区版Docker引擎，使用仓库方式安装(也可rpm包和脚本方式安装)
                - 设置仓库
                    + `sudo yum install -y yum-utils`
                    + `sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo`
                - 安装DOCKER ENGINE - COMMUNITY，最新版本：`sudo yum install docker-ce docker-ce-cli containerd.io -y`
    - 验证安装是否成功
        + `docker version`, ubuntu 16.04-xenial中，`Version: 17.09.1-ce`、`Go version:   go1.8.3`
        + 或 `docker info`，非root用户执行会报错，没有权限，可以把用户加到docker用户组中，按下面的操作：
    - Docker 需要用户具有 sudo 权限，为了避免每次命令都输入sudo，可以把用户加入 Docker 用户组
        + Docker的守护进程(daemon)是和Unix socket绑定，而不是TCP端口，默认情况下Unix socket属于root用户，其他用户需要sudo访问
        + 创建组`groupadd docker`， 用户添加到组 `sudo usermod -aG docker $USER`
        + 更新用户组：`newgrp docker` (或者退出登录重新登录，否则普通用户还是没权限)
            * 没权限会报错：`Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock`
    - Docker 是服务器-客户端架构。命令行运行docker命令的时候，需要本机有 Docker 服务。如果这项服务没有启动，可以用下面的命令启动
        + `sudo service docker start` 或 `sudo systemctl start docker`
    - 配置自启动
        + `sudo systemctl enable docker`
            * 取消自启动：`systemctl disable docker`

* Mac下报错`net/http: TLS handshake timeout`
    - Mac下安装docker完成后，执行示例：`docker run hello-world`，但是报错：
        + Unable to find image 'hello-world:latest' locally
        + docker: Error response from daemon: Get https://registry-1.docker.io/v2/: net/http: TLS handshake timeout
    - (尝试)生成证书，并没成功(为`https://registry.docker-cn.com`生成证书，最后curl发现链接并不通！)
        + `mkdir -p certs`
        + `openssl req \
              -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key \
              -x509 -days 365 -out certs/domain.crt`
        + `sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain domain.crt`
        + [Adding Self-signed Registry Certs to Docker & Docker for Mac](https://blog.container-solutions.com/adding-self-signed-registry-certs-docker-mac)
        + 生成CA：[Use self-signed certificates](https://docs.docker.com/registry/insecure/#using-self-signed-certificates#use-self-signed-certificates)
    - 国内镜像地址：`registry.docker-cn.com`
        + Docker Desktop -> Preferences->Daemon->Insecure registries，添加地址重启，并没有用
        + Registry mirrors中添加https不行，提示无认证，添加`http://registry.docker-cn.com`，还是不行。。
    - Docker Desktop启动时很卡，卸载之，
        + 找了一圈都是Desktop安装
        + 下载[安装包](https://download.docker.com/mac/stable/Docker.dmg)，发现貌似还是desktop mac，从垃圾堆找回来。。心态崩了崩了
        + [Mac平台上Docker安装与使用](https://blog.csdn.net/jiang_xinxing/article/details/58025417)
    - **解决方式**最后用了网易的镜像源：`http://hub-mirror.c.163.com`
        + Docker Desktop -> Preferences->Daemon->Insecure registries->Registry mirrors中添加该镜像源后重启
        + `curl -v http://registry.docker-cn.com`连接**不通**，折腾了那么久!!!
        + [国内 docker 仓库镜像对比](https://ieevee.com/tech/2016/09/28/docker-mirror.html)

### image文件

* image文件
    - Docker 把应用程序及其依赖，打包在 image 文件里面。只有通过这个文件，才能生成 Docker 容器。image 文件可以看作是容器的模板。Docker 根据 image 文件生成容器的实例。同一个 image 文件，可以生成多个同时运行的容器实例。
    - image 是二进制文件。实际开发中，一个 image 文件往往通过继承另一个 image 文件，加上一些个性化设置而生成。
    - `docker image ls` 列出本机的所有 image 文件。
    - `docker image rm [imageName]` 删除 image 文件
    - image 文件是通用的，一台机器的 image 文件拷贝到另一台机器，照样可以使用。
        + 一般来说，为了节省时间，我们应该尽量使用别人制作好的 image 文件，而不是自己制作。即使要定制，也应该基于别人的 image 文件进行加工，而不是从零开始制作。
        + 为了方便共享，image 文件制作完成后，可以上传到网上的仓库。Docker 的官方仓库 [Docker Hub](https://hub.docker.com/)，官方仓库是*最重要、最常用*的 image 仓库。

### 实例

* 仓库修改
    - 需要说明的是，国内连接 Docker 的官方仓库很慢，还会断线，需要将`默认仓库`改成*国内的镜像网站*。这里推荐使用官方镜像 `registry.docker-cn.com`。
        + 打开/etc/default/docker文件（需要sudo权限），在文件的底部加上一行。`DOCKER_OPTS="--registry-mirror=https://registry.docker-cn.com"`
        + 然后，重启 Docker 服务。`sudo service docker restart`，就会自动从镜像仓库下载 image 文件了
    - 修改参考：[Docker 微服务教程](http://www.ruanyifeng.com/blog/2018/02/docker-wordpress-tutorial.html)

* 实例
    - `docker image pull library/hello-world` 抓取 image 文件，library/hello-world是 image 文件在仓库里面的位置，其中library是 image 文件所在的组，hello-world是 image 文件的名字。
        + 由于 Docker 官方提供的 image 文件，都放在library组里面，所以它的是默认组，可以省略。因此，上面的命令可以写成：`docker image pull hello-world`
    - `docker image ls` / `docker images` 查看
    - `docker container run hello-world` 命令会从 image 文件，生成一个正在运行的容器实例。
        + 注意，`docker container run`命令具有自动抓取 image 文件的功能。如果发现本地没有指定的 image 文件，就会从仓库自动抓取。因此，前面的docker image pull命令并不是必需的步骤。
    - 有些容器不会自动终止，因为提供的是服务。
        + 比如，安装运行 Ubuntu 的 image，就可以在命令行体验 Ubuntu 系统。
            * `docker container run -it ubuntu bash`
        + 对于那些不会自动终止的容器，必须使用`docker container kill` 命令手动终止。
            * `docker container kill [containID]`
            * `docker ps -a ` 列出所有container，可以看到containerID

## Docker 常用命令

上面已经提到过一些：

* `docker ps`
    - 查看正在运行的容器实例
* `docker logs 容器名`
    - e.g. `docker logs mysql`
* `docker exec -it`
    - 进入容器
        + -i, --interactive[=false] 保持STDIN打开，即使没有连接
        + -t, --tty[=false]  分配一个伪 tty 设备
    - e.g. `docker exec -it mysql bash`
* `docker image`
    - 参考上面的章节记录(ls/rm/pull)

* 关于mysql容器相关的其他操作：
    - 启动一个进入到容器中的shell终端 `docker exec -it mysql1 bash`
        + 进入容器后操作和linux一致，mysql路径：/var/lib/mysql
    - 停止容器
        + `docker stop mysql1`
        + 会发送一个SIGTERM信号给mysqld进程，因此服务能优雅关闭
        + 注意，容器中的主进程停止的话，容器也会停止。 若mysqld停止，则包含其的容器也会停止
    - 再次启动容器 `docker start mysql1`
    - 重启容器 `docker restart mysql1`
    - 删除容器(两步，先停止再删除)
        + `docker stop mysql1`
        + `docker rm mysql1`
* 在运行docker容器时可以加如下参数来保证每次docker服务重启后容器也自动重启
    - `docker run --restart=always`
    - 如果已经启动了则可以使用如下命令
        + `docker update --restart=always <CONTAINER ID>`
* `docker run`
    - [Docker run 命令](https://www.runoob.com/docker/docker-run-command.html)
    - 创建一个新的容器并运行一个命令
    - e.g. `docker run --privileged --name=app -itd feisky/app:helloworld`
    - `--name[=NAME]`
        + 为容器指定一个名称；
    - `--privileged`
        + 赋予容器扩展特权(默认情况下容器是`unprivileged`非特权模式)，允许容器访问任何设备的权限
    - `-i` 或者 `--interactive`
        + 以交互模式运行容器，通常与 `-t` 同时使用
        + 即使没有连接也保持`STDIN`打开
    - `-t` 或 `--tty`
        + 为容器重新分配一个伪输入终端，通常与 `-i` 同时使用；
    - `-d` 或 `--detach`
        后台运行容器，并返回容器ID；
* `docker run`官网说明(其他指令可参考官网链接关联章节)
    - [Docker run reference](https://docs.docker.com/engine/reference/run/)
    - 基本形式：`docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]`
        + `:TAG` 可指定容器镜像版本，e.g. `docker run ubuntu:14.04`
        + `@DIGEST` 可按摘要指定容器镜像，e.g. `docker run alpine@sha256:9cacb71397b640eca97488cf08582ae4e4068513101088e9f96c9814bfda95e0 date`(alpine镜像摘要为sha256:9..)
    - 启动一个容器时，必须指定是*"前台模式"(foreground，默认)* 还是*"分离模式"(detached模式，也叫后台模式background)*
        + 若指定分离模式，则使用 `-d=true` 或 `-d`，分离模式下`根(root)进程`退出时，容器随即退出，除非设置了`--rm`选项
            * `-d`和`--rm`同时使用时，进程退出或守护进程退出时，容器即被移除(和退出有差别，退出加删除容器)
            * 不要传递`service x start`命令给分离的容器，root进程(`service x start`)结束返回后，容器随即就结束了
                - e.g. `docker run -d -p 80:80 my_image service nginx start`，导致的结果就是nginx启动之后，容器随即退出，并没有使用到nginx
        + 前台模式
            * `-t` 为容器重新分配一个伪输入终端
            * `-i` 以交互模式运行容器
            * `-a=[]`，可将Docker绑定到标准输入、输出、错误，e.g. `-a stdin -a stdout`
            * 注意，容器中PID为`1`的进程，Linux会特殊处理，会忽略任何信号的默认操作，因此`SIGINT`和`SIGTERM`信号并不会终止进程
    - 容器标识
        + 有三种方式来标识一个容器
            * `UUID`长标识符
                - e.g. `“f78375b1c487e03c9438c729345e54db9d20cfa2ac1fc3494b6eb60872e74778”`
                - UUID标识来自Docker守护进程(daemon)
            * UUID短标识符
                - e.g. `“f78375b1c487”`，取完成UUID前12位
            * 名称
                - e.g. `"myredis"`
                - 如果没有用`--name`选项指定名称，守护进程会生成一个随机字符串名称
    - `docker run --rm`
        + `--rm`指定退出容器时自动清理容器和文件系统(默认情况是不清理容器和数据的，便于查询上一次的最终状态)
        + 当设置`--rm`选项时，Docker也会移除容器关联的匿名卷，运行情况和 `docker rm -v my-container` 类似

* 时区问题
    - 遇到docker时间不一致，大多是因为默认时区没有设置导致，一般在宿主机上使用 date 命令看到的是 CTS 时间，进入docker后使用 date 命令查看的是 UTC 时间。
        + CTS： China Standard Time，UTC+8:00 中国沿海时间（北京时间）
        + UTC： Universal Time Coordinated 世界协调时间
    - 设置方法：
        + 1、docker run 的时候增加环境变量 `-e TZ=Asia/Shanghai`（这个有时候不太好使）
        + 2、添加volumes映射 /etc/localtime 映射到 /etc/localtime（可靠）
            * `docker run -v /etc/localtime:/etc/localtime <IMAGE:TAG>`
        + 3、如果是你的镜像是自己Dockerfile编译的，那么在你的Dockerfile中添加
            * `RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai' >/etc/timezone`
    - [Docker时间不一致，时区设置](https://blog.csdn.net/catoop/article/details/89737861?depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-2&utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-2)
    - 宿主机上设置时区，可(到容器中这样操作后重启容器也可)
        + a. 直接手动创建软链接 `ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime`
        + b. `cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime`
* `docker build`
    - build选项
        + `-f`可指定dockerfile文件路径，e.g. `docker build -f /path/to/a/Dockerfile .`
        + `-t`可指定tag(可包括存储库和版本)，e.g. `docker build -t shykes/myapp .`
            * 同时指定多个存储库：`docker build -t shykes/myapp:1.0.2 -t shykes/myapp:latest .`
        + 从Dockerfile进行编译，`docker build .`是docker daemon运行的(build第一步将内容发给daemon)，而不是客户端
    - Dockerfile
    - 参考自：[Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
* `docker commit`
    - `docker commit xxxxx` 会产生悬挂镜像，一般悬挂镜像并不总是我们需要的，会浪费磁盘空间
        + `docker images --filter "dangling=true" -q` 查看悬挂镜像，`|xargs docker rmi` 可用管道删除
            * `-q` 只输出image ID
            * `--no-trunc` 输出完整image ID
* 导入镜像
    - `docker import`
        + 用于导入包含根文件系统的归档，并将之变成Docker镜像
    - `docker load`
        + 一般只用于通过`docker save`导出的镜像
## 官网文档

* Quickstart
    - 安装并运行hello-world镜像测试
        + [Orientation and setup](https://docs.docker.com/get-started/)
    - 编译运行容器镜像
        + [Build and run your image](https://docs.docker.com/get-started/part2/)
        + 创建一个供应用程序运行的docker镜像，其提供程序需要的私有的文件系统
        + 通过示例学习容器化一个应用(Node.js写的一个简单的公告栏应用)
            * `git clone https://github.com/dockersamples/node-bulletin-board`
            * `cd node-bulletin-board/bulletin-board-app`
            * 查看Dockerfile，`FROM`,`WORKDIR`,`COPY`,`RUN`,`EXPOSE`,`CMD`,`COPY`
                - `FROM`,`WORKDIR`,`COPY`,`RUN`,`COPY` 用来构建镜像的文件系统
                - `CMD`指定一些运行一个容器需要的元数据，`EXPOSE`指定监听端口
                - 示例中是一个很好的组织Dockerfile的方式，总是从`FROM`开始，后续步骤是构建私有文件系统，然后指定一些元数据
                - 关于Dockerfile可参考(查看本笔记中`docker build`章节)：[Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
            * 编译
                - `docker build --tag bulletinboard:1.0 .`
                - 编译成功会显示："Successfully tagged bulletinboard:1.0"
            * 运行
                - `docker run --publish 8000:8080 --detach --name bb bulletinboard:1.0`
                    + `--publish 8000:8080`(同`-p`) 要求docker将传入给主机端口8000的流量转发给容器中的8080端口
                    + `--detach`(同`-d`) 后台运行
                    + `--name bb` 容器命名为bb
            * 删除容器
                - `docker rm --force bb`
                    + `--force`会移除容器(`docker ps -a`就看不到了)
                    + 若要删除镜像，则使用：`docker rmi`，注意`docker rm`删除容器，`rmi`删除镜像
                - 如果只是停止容器(而不删除)，则用 `docker stop bb`(`docker ps -a`还能看到)
            * 下一步是分享镜像到Docker Hub，便于任何机器来下载和运行
    - 在Docker Hub上分享镜像
        + [Share images on Docker Hub](https://docs.docker.com/get-started/part3/)
        + 注册Docker Hub账户，[Create a Docker ID](https://hub.docker.com/signup)
        + 创建仓库，登陆Docker Hub之后->Repositories标签下->Create Repository->创建仓库名称e.g.`bulletinboard`
        + 设置镜像的命名空间 `docker tag bulletinboard:1.0 xdargs/bulletinboard:1.0` (xdargs个人Docker ID)
        + 推送镜像到Docker Hub `docker push xdargs/bulletinboard:1.0`
        + 访问Docker Hub的仓库就可以看到新镜像了
        + 其他机器上pull使用容器即可
* [Develop with Docker](https://docs.docker.com/develop/)
    - 减小编译出来的镜像大小是最常见的一个挑战，常见一个方式是使用一个Dockerfile用于开发(包括编译应用需要的所有环境和组件)，另一个瘦身的Dockerfile用于产品发布(只包括需要运行的程序和运行需要的组件)
        + 不过维护两个Dockerfile不是最理想的，链接中的示例展示了两个Dockerfile，Dockerfile.build和Dockerfile

## Docker镜像操作

* 把 docker 镜像导出到文件
    - `docker save -o kube-proxy_1.17.3.tar k8s.gcr.io/kube-proxy:v1.17.3`
* 导入 docker 镜像文件
    - `docker load -i kube-proxy_1.17.3.tar`

* [将 Docker 镜像体积减小 99%](https://yq.aliyun.com/articles/751376)
    - [Docker之操作系统Alpine](https://blog.csdn.net/bbwangj/article/details/81088231)
    - [CGO_ENABLED环境变量对Go静态编译机制的影响](https://johng.cn/cgo-enabled-affect-go-static-compile/)
        + 纯静态编译

---

此前有些笔记是很早之前的了，很久没用基本没太多实用价值，暂时不删除。

后续记录实践过程中碰到的问题。

## 记一次CentOS新环境安装Docker

参考：[Docker — 从入门到实践，CentOS 安装 Docker](https://docker-practice.github.io/zh-cn/install/centos.html)

安装：卸载老的(注意会把依赖删掉，有可能其他服务依赖也会受影响，后续尝试`--nodeps`?)，安装`yum-utils`并使用`yum-config-manager`安装，使用阿里云国内源
(CentOS8用podman替代docker，可以删除换成docker)

```sh
yum erase podman buildah
yum install docker-ce docker-ce-cli containerd.io
systemctl start docker
systemctl enable docker
```

设置国内docker镜像源，Docker的配置文件通常位于`/etc/docker/daemon.json`。如果该文件不存在则创建
    具体参考：[镜像加速器](https://docker-practice.github.io/zh-cn/install/mirror.html)

设置源，由于镜像服务可能出现宕机，建议同时配置多个镜像。(注意需要带上`https://`，而ping检查联通性时不带)
当前实际配置如下：

```sh
{    
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"                                                              
  ]  
}  
```

`systemctl daemon-reload`
`systemctl restart docker`

`docker info`查看是否生效

```sh
[root@desktop-mme7h3a ➜ /root ]$ docker info
Client: Docker Engine - Community
 Version:    26.1.3
 Context:    default
 Debug Mode: false
 ...
Registry Mirrors:
  https://docker.m.daocloud.io/
  https://hub-mirror.c.163.com/
  https://mirror.baidubce.com/
 Live Restore Enabled: false
```

碰到的问题：

ping 127.0.0.1也报错：

```sh
[root@desktop-mme7h3a ➜ /root ]$ ping 127.0.0.1       
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
ping: sendmsg: Operation not permitted
ping: sendmsg: Operation not permitted
```

strace查看创建socket就报权限问题

```sh
[root@desktop-mme7h3a ➜ /root ]$ strace -yy ping 127.0.0.1
execve("/usr/sbin/ping", ["ping", "127.0.0.1"], 0x7fff1f4cf790 /* 43 vars */) = 0
...
socket(AF_INET, SOCK_DGRAM, IPPROTO_ICMP) = -1 EACCES (Permission denied)
socket(AF_INET, SOCK_RAW, IPPROTO_ICMP) = 3<RAW:[0.0.0.0:1]>
socket(AF_INET6, SOCK_DGRAM, IPPROTO_ICMPV6) = -1 EACCES (Permission denied)
```

怀疑之前设置用户组的问题，先不管，重启解决了(另外之前修改了hostname，重启后才生效了)

```sh
# 重启前
[root@desktop-mme7h3a ➜ /root ]$ id
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

# 重启后
[root@xdlinux ➜ /root ]$ id
uid=0(root) gid=0(root) groups=0(root),982(docker)
```

验证安装配置正常：

```sh
[root@xdlinux ➜ /root ]$ docker run --rm hello-world                        
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
c1ec31eb5944: Pull complete 
Digest: sha256:d1b0b5888fbb59111dbf2b3ed698489c41046cb9d6d61743e37ef8d9f3dda06f
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

## 测试程序制作镜像

```sh
# 使用一个带有g++的base image  
FROM gcc:latest  
  
# 使用 WORKDIR 指令可以来指定工作目录（或者称为当前目录），以后各层的当前目录就被改为指定的目录
# 如该目录不存在，WORKDIR 会帮你建立目录。
# 设置工作目录  
WORKDIR /app  
  
# COPY 指令将从构建上下文目录中的文件/目录复制到新的一层的镜像内的目的路径位置。
# Dockerfile和要拷贝的内容需要在相同的根目录
# 将你的C++代码复制到Docker容器中  
COPY server.cpp client.cpp make.sh /app/

# ADD 命令，和COPY类似，在其基础上加了一些功能
# 如果源路径是一个tar/gzip//bzip2/xz包，则ADD会自动解压
# 在 Docker 官方的 Dockerfile 最佳实践文档 中要求，尽可能的使用 COPY，因为 COPY 的语义很明确
  
# RUN 指令用于在镜像构建过程中执行命令。这些命令会在构建镜像时运行，并且它们的结果会被提交到新的镜像层中。
# 编译你的C++代码（假设你有一个名为main.cpp的文件）  
RUN bash make.sh 

# CMD 设置容器启动时默认执行的命令及其参数，这个命令会在容器启动时运行，而且可以被 docker run 命令后面的参数所覆盖。
# 运行编译后的程序  
CMD ["./server"]
```

1、构建：
`docker build -t test-tcp-connect .`

gcc镜像一直拉不下来

2、设置阿里云镜像加速器：

参考：[设置阿里云镜像加速器](https://developer.aliyun.com/article/1402795?spm=a2c6h.14164896.0.0.3d4747c546qVzc&scm=20140722.S_community@@%E6%96%87%E7%AB%A0@@1402795._.ID_1402795-RL_%E9%95%9C%E5%83%8F%E5%8A%A0%E9%80%9F%E5%99%A8-LOC_search~UND~community~UND~item-OR_ser-V_3-P0_0)

阿里云，搜索“镜像加速器”，要登录获取加速器地址。上面还有设置方法
[镜像加速器](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors?spm=a2c6h.12873639.article-detail.12.799a4e6dYofFto)

碰到docker重启不起来。参考这篇解决：
[创建默认“网桥”网络时出错:无法创建网络(docker0)：与网络(Docker0)冲突:网络具有相同的网桥名称](https://cloud.tencent.com/developer/ask/sof/105054756)

```sh
rm -rf /var/docker/network/*
mkdir /var/docker/network/files
systemctl start docker
```

3、设置后重新build，成功。

```sh
# 编译镜像
[root@xdlinux ➜ tcp_connect git:(main) ✗ ]$ docker build -t test-tcp-connect .
[+] Building 62.8s (9/9) FINISHED                                                           docker:default
 => [internal] load build definition from Dockerfile                                                  0.0s
 => => transferring dockerfile: 369B                                                                  0.0s
 => [internal] load metadata for docker.io/library/gcc:latest                                        17.0s
 => [internal] load .dockerignore                                                                     0.0s
 => => transferring context: 2B                                                                       0.0s
 => [1/4] FROM docker.io/library/gcc:latest@sha256:084eaedf8e3c51f3db939ad7ed2b1455ff9ce4705845a014  43.4s
 => => resolve docker.io/library/gcc:latest@sha256:084eaedf8e3c51f3db939ad7ed2b1455ff9ce4705845a014f  0.0s
 => => sha256:9b829c73b52b92b97d5c07a54fb0f3e921995a296c714b53a32ae67d19231fcd 5.15MB / 5.15MB        1.4s
 => => sha256:084eaedf8e3c51f3db939ad7ed2b1455ff9ce4705845a014fb9fe5577b35c901 1.43kB / 1.43kB        0.0s
 => => sha256:f7ea55625e097107d176d06641b637eeae64c606dbf9fa2938872e4c0aca15bc 2.22kB / 2.22kB        0.0s
 => => sha256:cb5b7ae361722f070eca53f35823ed21baa85d61d5d95cd5a95ab53d740cdd56 10.87MB / 10.87MB      5.6s
 => => sha256:709e60f7d3e3c4c83db820a7edc55c7fc50b52d18870b62165aef3bd38d97f64 9.39kB / 9.39kB        0.0s
 => => sha256:0e29546d541cdbd309281d21a73a9d1db78665c1b95b74f32b009e0b77a6e1e3 54.92MB / 54.92MB     26.1s
 => => sha256:6494e4811622b31c027ccac322ca463937fd805f569a93e6f15c01aade718793 54.57MB / 54.57MB      7.2s
 => => sha256:6f9f74896dfa93fe0172f594faba85e0b4e8a0481a0fefd9112efc7e4d3c78f7 196.51MB / 196.51MB   39.7s
 => => sha256:9e02d5ba8945fff0903d608586865f4a30fccd4a5d038ee2366339be3052676b 15.43kB / 15.43kB      7.7s
 => => sha256:f26bedb21a7b9d02daceea55babc6c2bbaf0e30c4d6fdb3b3f1a81e0fd810ed6 127.26MB / 127.26MB   30.2s
 => => extracting sha256:0e29546d541cdbd309281d21a73a9d1db78665c1b95b74f32b009e0b77a6e1e3             0.9s
 => => sha256:6584fc653633ae1641051770b2e9f7c13f3be570204b2875c343c27b312c9fbf 10.10kB / 10.10kB     26.6s
 => => sha256:bae0ff2cc62a65ad7630a9e39e71081c85e5669bfa74cdc5c971385ec4daea25 1.89kB / 1.89kB       27.1s
 => => extracting sha256:9b829c73b52b92b97d5c07a54fb0f3e921995a296c714b53a32ae67d19231fcd             0.1s
 => => extracting sha256:cb5b7ae361722f070eca53f35823ed21baa85d61d5d95cd5a95ab53d740cdd56             0.1s
 => => extracting sha256:6494e4811622b31c027ccac322ca463937fd805f569a93e6f15c01aade718793             0.9s
 => => extracting sha256:6f9f74896dfa93fe0172f594faba85e0b4e8a0481a0fefd9112efc7e4d3c78f7             2.1s
 => => extracting sha256:9e02d5ba8945fff0903d608586865f4a30fccd4a5d038ee2366339be3052676b             0.0s
 => => extracting sha256:f26bedb21a7b9d02daceea55babc6c2bbaf0e30c4d6fdb3b3f1a81e0fd810ed6             1.0s
 => => extracting sha256:6584fc653633ae1641051770b2e9f7c13f3be570204b2875c343c27b312c9fbf             0.0s
 => => extracting sha256:bae0ff2cc62a65ad7630a9e39e71081c85e5669bfa74cdc5c971385ec4daea25             0.0s
 => [internal] load build context                                                                     0.0s
 => => transferring context: 89B                                                                      0.0s
 => [2/4] WORKDIR /app                                                                                1.3s
 => [3/4] COPY server.cpp client.cpp make.sh /app/                                                    0.1s
 => [4/4] RUN bash make.sh                                                                            0.9s
 => exporting to image                                                                                0.1s
 => => exporting layers                                                                               0.0s
 => => writing image sha256:1a808f6d5ad196ebffa9615ba404fcaabda7384aad6e3d6721d9014710d0b0d1          0.0s
 => => naming to docker.io/library/test-tcp-connect                                                   0.0s

# 编译成功：test-tcp-connect
[root@xdlinux ➜ tcp_connect git:(main) ✗ ]$ docker images
REPOSITORY         TAG       IMAGE ID       CREATED          SIZE
test-tcp-connect   latest    1a808f6d5ad1   47 seconds ago   1.23GB
alpine             latest    a606584aa9aa   2 days ago       7.8MB
testnginx          latest    2d3d2bb36614   7 days ago       188MB
hello-world        latest    d2c94e258dcb   13 months ago    13.3kB
```

`docker run -d -p 8080:8080 --name my-test-tcp-connect-container test-tcp-connect`

```sh
# 创建一个docker容器
[root@xdlinux ➜ tcp_connect git:(main) ✗ ]$ docker run -d -p 8080:8080 --name my-test-tcp-connect-container test-tcp-connect
f52f3b53e7c5be0dd16d132b169c2f952becf0b9bfd86580cf9e6a794ee40fee
# 查看监听
[root@xdlinux ➜ tcp_connect git:(main) ✗ ]$ netstat -anp|grep 8080
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      8819/docker-proxy   
tcp6       0      0 :::8080                 :::*                    LISTEN      8826/docker-proxy
```

客户端可通过服务端ip和8080访问服务。
