# 配置yum 源

* yum源配置文件实际内容如下

```sh
[centos-local-yum]
name=CentOS local
baseurl=file:///home/iso/mnt
enabled=1
gpgcheck=1
gpgkey=file:///home/iso/mnt/RPM-GPG-KEY-CentOS-7
```

* 若有内部网络的yum源

`mv /etc/yum.repos.d/* /etc/yum.repos.d/repobak/`
`cp ./http.repo /etc/yum.repos.d/`

```
[http]
name="http yum"
baseurl=http://xx.xx.xx.xx/centos/7/os/x86_64/
enabled=1
gpgcheck=0
```

#### 6.9yum源(DVD1、DVD2)

2、上传 Centos 镜像到服务器，挂载 Centos 镜像文件

mount -o loop /mnt/iso/CentOS-6.5-x86_64-bin-DVD1.iso /mnt/dvd1
mount -o loop /mnt/iso/CentOS-6.5-x86_64-bin-DVD2.iso /mnt/dvd2

3、拷贝文件
首先, 拷贝第一张DVD中的所有文件到 /mnt/dvd3 目录下，然后, 只拷贝第二张 DVD 中 Packages 目录下的所有RPM文件到 /mnt/dvd3/Packages 目录下

cp -av /mnt/dvd1 /mnt/dvd3
cp -v /mnt/dvd2/Packages/*.rpm /mnt/dvd3/Packages/

4、合并TRANS.TBL

将DVD2中TRANS.TBL的信息追加到DVD1中TRANS.TBL后面, 并排序保存

cat /mnt/dvd2/Packages/TRANS.TBL >> /mnt/dvd3/Packages/TRANS.TBL
mv /mnt/dvd3/Packages/{TRANS.TBL,TRANS.TBL.BAK}
sort /mnt/dvd3/Packages/TRANS.TBL.BAK > /mnt/dvd3/Packages/TRANS.TBL
rm -rf /mnt/dvd3/Packages/TRANS.TBL.BAK

dvd3已经是合并后的文件了，可以用作本地源和做成ISO使用

### 配置ftp服务访问本地yum源

* 配置ftp服务访问本地yum源(虚拟机里可以通过ftp方式配置宿主机的本地源)
* [利用FTP服务搭建本地yum源](https://blog.csdn.net/weixin_43458720/article/details/88728816)
* 服务器配置
    - 主机安装ftp：`yum install vsftpd -y`
    - 添加对本地yum源路径的root匿名访问：
        + `vi /etc/vsftpd/vsftpd.conf`
        + 新增`anon_root=/home/iso/` (即挂载目录的上级目录)
    - 启动ftp并设置开机启动
        + `systemctl start vsftpd`
        + `systemctl enable vsftpd`
* 客户机配置
    - 配置客户端节点yum源文件
        + 备份删除原文件 `cp -r /etc/yum.repos.d /etc/yum.repos.d_bak && rm -rf /etc/yum.repos.d/*`
        + 新建配置文件`vi /etc/yum.repos.d/CentOS-local.repo`，配置如下
    - 测试
        + `yum repolist`查看可看到个数 (`yum list`结果个数太多时，执行比较慢)
        + 安装vim，`yum install vim -y`
        + 安装网络管理工具(包含ifconfig、route、netstat等等)，`yum install -y net-tools`

```sh
# 客户端节点yum源配置文件 CentOS-local.repo
# 其中 xx.xx.xx.xx 为配置了yum本地源的地址
[centos-ftp-yum]
name=CentOS ftp
baseurl=ftp://xx.xx.xx.xx/mnt
enabled=1
gpgcheck=1
gpgkey=ftp://xx.xx.xx.xx/mnt/RPM-GPG-KEY-CentOS-7
```

### 配置http服务访问本地yum源

[基于http的yum源](https://blog.csdn.net/bayin4937/article/details/100949916)

服务端：
1. 本地挂载
2. 修改本地yum(用于安装http服务)
3. 安装httpd并启动 `yum install httpd`，`service httpd start`
    若端口被占用，可修改：/etc/httpd/conf/httpd.conf
4. 创建http挂载，到`/var/www/html`创建目录并挂载
    `mkdir -p /var/www/html/euler_2203/os/x86_64/`(注意euler_2203、os、x86_64的权限都检查下，需要755) 并mount镜像
        drwxr-x--- 3 root root 4.0K Apr 13 16:46 euler_2203 (euler上mkdir其他组没有访问权限)
        -rw-r----- 1 root         root      0 Apr 13 17:20 temp (手动创建的文件，其他用户没权限)
    `mount -o ro /home/qxd/openEuler-22.03-LTS-x86_64-dvd.iso /var/www/html/euler_2203/os/x86_64/`

```sh
# /home/qxd/euler_iso为本地挂载路径
[local-yum]
name=local
baseurl=file:///home/qxd/euler_iso
enabled=1
gpgcheck=0
```

```sh
# 欧拉本地yum源，一键操作，需提前拷贝好镜像包
mnt_dir=/home/qxd/euler_iso
iso_path=/home/qxd/openEuler-22.03-LTS-x86_64-dvd.iso

mkdir -p $mnt_dir
mount -o ro $iso_path $mnt_dir
mv /etc/yum.repos.d/openEuler.repo /etc/yum.repos.d/openEuler.repo_bak
cat > ./local.repo << EOF
[local-yum]
name=local
baseurl=file://$mnt_dir
enabled=1
gpgcheck=0
EOF
yum clean all
yum makecache
```

客户端：

```sh
# http挂载
[http-yum]
name=http
baseurl=http://xx.xx.xx.xx:88/euler_2203/os/x86_64/
enabled=1
gpgcheck=0
```