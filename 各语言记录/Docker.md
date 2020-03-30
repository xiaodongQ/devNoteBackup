# docker

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
    - 需要说明的是，国内连接 Docker 的官方仓库很慢，还会断线，需要将`默认仓库`改成*国内的镜像网站*。这里推荐使用官方镜像 registry.docker-cn.com 。
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
