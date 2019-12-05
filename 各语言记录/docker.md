# docker

[Docker 入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)

## 背景

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

## Docker用途

* Docker用途
	- Docker 的主要用途，目前有三大类。
	- 1. 提供一次性的环境。
		+ 比如，本地测试他人的软件、持续集成的时候提供单元测试和构建的环境。
	- 2. 提供弹性的云服务。
		+ 因为 Docker 容器可以随开随关，很适合动态扩容和缩容。
	- 3. 组建微服务架构。
		+ 通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构。

## Docker安装

* Docker安装
	- Docker 是一个开源的商业产品，有两个版本：社区版（Community Edition，缩写为 CE）和企业版（Enterprise Edition，缩写为 EE）。企业版包含了一些收费服务，个人开发者一般用不到。下面的介绍都**针对社区版**。
	- Ubuntu
		+ [Get Docker Engine - Community for Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
		+ 要求Ubuntu系统版本
			* Disco 19.04、Cosmic 18.10、Bionic 18.04 (LTS)、Xenial 16.04 (LTS)
			* 查看自己Ubuntu虚拟机的版本：
				- `lsb_release -a`，可以看到相关行：`Release:	16.04`、`Codename:	xenial`
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
			* 开始安装(不设置仓库进行安装会找不到docker的包)
				- `sudo apt-get update`
				- `sudo apt-get install docker-ce docker-ce-cli containerd.io`
					+ 执行由于部分库版本不满足，需要升级ubuntu，虚拟机里一直升级失败，所以用指定版本的方式升级
				-  查看当前ubuntu版本能安装的版本
					+ 查看 docker-ce 满足版本 `apt-cache madison docker-ce`
						* 尝试列表里很多新一点的版本都有部分库版本不满足要求，最终选择了：17.09.1~ce-0~ubuntu
						* 使用 `sudo apt-get install docker-ce=17.09.1~ce-0~ubuntu`
					+ 查看 docker-ce-cli 满足版本 `apt-cache madison docker-ce-cli`
						* `sudo apt-get install docker-ce-cli=5:18.09.0~3-0~ubuntu-xenial` 安装时会卸载docker-ce。。。暂时不安装
					+ 查看 containerd.io 满足版本`apt-cache madison containerd.io`
						* `sudo apt-get install containerd.io=1.2.0~beta.2-1`，也要apt-cache madison查看能安装的版本。。。

	- 验证安装是否成功
		+ `docker version`, ubuntu 16.04-xenial中，`Version:      17.09.1-ce`、`Go version:   go1.8.3`
		+ 或 `docker info`，非root用户执行会报错，没有权限，可以把用户加到docker用户组中，按下面的操作
	- Docker 需要用户具有 sudo 权限，为了避免每次命令都输入sudo，可以把用户加入 Docker 用户组
		+ `sudo usermod -aG docker $USER`
		+ 更新用户组：`newgrp docker` (否则普通用户还是没权限)
			* 没权限会报错：Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock
	- Docker 是服务器----客户端架构。命令行运行docker命令的时候，需要本机有 Docker 服务。如果这项服务没有启动，可以用下面的命令启动
		+ `sudo service docker start` 或 `sudo systemctl start docker`

## image文件

* image文件
	- Docker 把应用程序及其依赖，打包在 image 文件里面。只有通过这个文件，才能生成 Docker 容器。image 文件可以看作是容器的模板。Docker 根据 image 文件生成容器的实例。同一个 image 文件，可以生成多个同时运行的容器实例。
	- image 是二进制文件。实际开发中，一个 image 文件往往通过继承另一个 image 文件，加上一些个性化设置而生成。
	- `docker image ls` 列出本机的所有 image 文件。
	- `docker image rm [imageName]` 删除 image 文件
	- image 文件是通用的，一台机器的 image 文件拷贝到另一台机器，照样可以使用。
		+ 一般来说，为了节省时间，我们应该尽量使用别人制作好的 image 文件，而不是自己制作。即使要定制，也应该基于别人的 image 文件进行加工，而不是从零开始制作。
		+ 为了方便共享，image 文件制作完成后，可以上传到网上的仓库。Docker 的官方仓库 [Docker Hub](https://hub.docker.com/) 是最重要、最常用的 image 仓库。

## 实例

* 仓库修改
	- 需要说明的是，国内连接 Docker 的官方仓库很慢，还会断线，需要将默认仓库改成国内的镜像网站。这里推荐使用官方镜像 registry.docker-cn.com 。
		+ 打开/etc/default/docker文件（需要sudo权限），在文件的底部加上一行。`DOCKER_OPTS="--registry-mirror=https://registry.docker-cn.com"`
		+ 然后，重启 Docker 服务。`sudo service docker restart`，就会自动从镜像仓库下载 image 文件了
	- 修改参考：[Docker 微服务教程](http://www.ruanyifeng.com/blog/2018/02/docker-wordpress-tutorial.html)

* hello-world
	- `docker image pull library/hello-world` 抓取 image 文件，library/hello-world是 image 文件在仓库里面的位置，其中library是 image 文件所在的组，hello-world是 image 文件的名字。
		+ 由于 Docker 官方提供的 image 文件，都放在library组里面，所以它的是默认组，可以省略。因此，上面的命令可以写成：`docker image pull hello-world`
	- `docker image ls` 查看
	- `docker container run hello-world` 命令会从 image 文件，生成一个正在运行的容器实例。
		+ 注意，`docker container run`命令具有自动抓取 image 文件的功能。如果发现本地没有指定的 image 文件，就会从仓库自动抓取。因此，前面的docker image pull命令并不是必需的步骤。