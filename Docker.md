[toc]

# CentOS7-Docker 安装前环境准备

```sh
# 下载阿里源
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
# 下载Docker专用源
wget -O /etc/yum.repos.d/docker-ce.repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 更新源
yum clean all && yum makecache
# 安装必要软件
yum install -y bash-completion vim vim-common  wget net-tools tree lrzsz expect  nc nmap  dos2unix htop iftop iotop unzip telnet sl psmisc nethogs glances bc ntpdate openldap-devel
# 查看内核版本是否大于 3.10
uname  -r
# 打开内核流量转发功能
cat <<EOF > /etc/sysctl.d/docker.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.conf.default.rp_filter = 0
net.ipv4.conf.all.rp_filter = 0
net.ipv4.ip_forward = 1
EOF
# 重启内核参数
modprobe br_netfilter
sysctl -p /etc/sysctl.d/docker.conf
# 查看 docker 列表及版本号
yum list docker-ce --showduplicates | sort -r
```

# CentOS7-Docker 安装 、卸载 

```sh
# 安装 docker
yum install -y docker-ce

# 卸载 docker 
yum remove -y docker-ce

# 配置镜像加速器
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://qokc1a1i.mirror.aliyuncs.com"]
}
EOF


# 检查配置是否生效
docker info
```

# Ubuntu-Docker 安装前环境准备

```sh
# 首先，更新软件包索引，并且安装必要的依赖软件，来添加一个新的 HTTPS 软件源：
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
# 使用下面的 curl 导入源仓库的 GPG key：
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# 将 Docker APT 软件源添加到你的系统：
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
# 现在，Docker 软件源被启用了，你可以安装软件源中任何可用的 Docker 版本。
# 01.想要安装 Docker 最新版本，运行下面的命令。如果你想安装指定版本，跳过这个步骤，并且跳到下一步。
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
# 02.想要安装指定版本，首先列出 Docker 软件源中所有可用的版本：
apt-get list -a docker-ce
```

# Ubuntu-Docker 安装、卸载

```sh
# 通过在软件包名后面添加版本=<VERSION>来安装指定版本：
sudo apt-get install -y docker-ce=<VERSION> docker-ce-cli=<VERSION> containerd.io
# 一旦安装完成，Docker 服务将会自动启动。你可以输入下面的命令，验证它：
sudo systemctl status docker
# 如果你想阻止 Docker 自动更新，锁住它的版本：
sudo apt-mark hold docker-ce
# 卸载 Docker：
sudo apt-get purge docker-ce
sudo apt-get autoremove
```

# Docker 服务相关命令

```sh
# 查看 docker 是否启动
systemctl is-active docker
# 重新加载系统 daemon 控制器
systemctl daemon-reload
# 启动 docker
  systemctl start docker
# 停止 docker 
systemctl stop docker
# 重启 docker
systemctl restart docker
# 查看 docker 启动状态
systemctl status docker
# 设置开机启动 docker
systemctl enable docker.service
```

# Docker 镜像相关命令

```sh
# 搜索镜像
docker search 镜像名
# 查看镜像
docker images 
# 查看所有镜像的ID
docker images -q
# 获取镜像
docker pull 镜像名:[版本号]
# 镜像列表
docker images
# 本地镜像删除
docker rmi 镜像名｜镜像ID
# 本地镜像全部删除
docker rmi `docker images -q`|$(docker images -q)
# 镜像保存
docker sava 镜像名
```



# Docker 容器管理命令

### 查看所有容器

```sh
docker ps   							#	查看正在运行的容器
docker ps -a							# 查看所有容器
docker inspect 容器名			# 查看容器信息
```

### 创建容器 

```sh
docker run
					 -i	 		# 保持和 docker 容器内的交互，启动容器时，运行的命令结束后，容器依然存活(没有退出)
					 -t 		# 为容器的标准输入虚拟一个终端(跟 -i 一起使用)
					 -d 		# 后台运行
					 --rm 	# 容器在启动后，程序停止就销毁
					 --name # 给容器起个自定义的名称
					 -p			# 宿主机端口映射:内部端口		
           -v     # 数据卷映射主机目录:容器目录
docker run -id --rm -name 自定义镜像名 -p 映射端口:内部端口  -v 数据卷主机目录:容器目录 要启动的镜像
```

### 启动容器

```sh
docker start 容器名｜容器ID 											# 开启容器
docker container start 容器名｜容器ID 						# 开启容器
docker start `docker ps -aq`|$(docker ps -aq) 	# 开启所有容器
```

### 停止(关闭)容器

```sh
docker stop 容器名｜容器ID 												# 停止正在运行的容器
docker stop `docker ps -aq`|$(docker ps -aq) 		# 停止所有正在运行的容器
```

### 删除容器

```sh
docker rm 容器名｜容器ID 													# 删除容器
docker rm `docker ps -aq`|$(docker ps -aq) 			# 删除全部容器
```

### 进入容器

#### 1.1、宿主机关闭防火墙

```sh
# 查看防火墙开放的端口
firewall-cmd --list-ports
# 开启防火墙 
systemctl start firewalld
# 选择关闭防火墙 
systemctl stop firewalld
# 禁止防火墙开机启动
systemctl disable firewalld.service
# 重启防火墙
firewall-cmd --reload
```

#### 1.2、宿主机开放防火墙端口

```sh
# 查看防火墙开放的端口
firewall-cmd --list-ports
# 开放80端口
firewall-cmd --zone=public --add-port=80/tcp --permanent
# 关闭80端口
firewall-cmd --zone=public --remove-port=80/tcp --permanent
# 重启防火墙
firewall-cmd --reload
```

#### 2、进入容器指令

```sh
docker exec -it 容器名｜容器ID 	bash
```

#### 3、退出容器指令

```sh
exit
```

### 互相拷贝文件

#### 1、宿主机内容拷贝docker至容器中

```sh
docker cp 文件内容  容器名称:容器路径
docker cp 1.txt nginx01:/root
```

#### 2、将 docker 容器内容复制到宿主机中

```sh
docker cp 容器名称:/root/1.txt 宿主机路径
```

# Docker 部署 MySQL

```sh
# 搜索 mysql 镜像
docker search mysql 
# 拉取 mysql 镜像
docker pull mysql
# 创建 mysql 映射卷
mkdir ~/mysql
# 切换进 mysql 目录里面
cd ~/mysql 
# 开启 mysql 容器
docker run -id --rm \
-p 3306:3306 \
--name mysql \
-v $PWD/conf:/etc/mysql/conf.d \
-v $PWD/logs:/logs \
-v $PWD/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=a6230341 \
mysql
# 进入 mysql 
docker exec -it mysql /bin/bash
# 登陆 mysql
mysql -uroot -pa6230341
# 退出 mysql
exit
# 提出容器
exit
# 外部链接的信息为：宿主机IP 端口号为映射的端口
```

# Docker 部署 Tomcat

```sh
# 搜索 Tomcat 镜像
docker search tomcat 
# 拉取 Tomcat 镜像
docker pull tomcat
# 创建 Tomcat 映射卷
mkdir ~/tomcat
# 切换进 Tomcat 目录里面
cd ~/tomcat 
# 开启 Tomcat 容器
docker run -id --rm \
--name tomcat \
-p 8080:8080 \
-v $PWD:/usr/local/tomcat/webapps \
tomcat
```

# Docker 部署 Nginx

```sh
# 搜索 Nginx 镜像
docker search nginx 
# 拉取 Nginx 镜像
docker pull nginx
# 创建 Nginx 映射卷
mkdir -p ~/nginx/conf/conf.d 
cd ~/nginx/conf
# 在 ～/nginx/conf 目录下创建 nginx.conf 文件，黏贴以下内容：
vim nginx.conf
```

```sh
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}

```

```sh
# 在 ～/nginx/conf/conf.d 目录下创建 default.conf 文件，黏贴以下内容：
vim  ～/nginx/conf/conf.d/default.conf
```

```sh
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;
    
    #access_log  /var/log/nginx/host.access.log  main;
    
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    
    #error_page  404              /404.html;
    # redirect server error pages to the static page /50x.html
    
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
    
    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}
    
    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}
    
    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```



# 

```sh
# 开启 Nginx 容器
docker run -id  \
--name nginx \
-p 80:80 \
-v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf \
-v $PWD/conf/conf.d:/etc/nginx/conf.d \
-v $PWD/logs:/var/log/nginx \
-v $PWD/html:/usr/share/nginx/html \
nginx:latest
```

# Docker 镜像制作

## DockerFile 的指令

```sh
FROM					# 基础镜像
MAINTAINER		# 镜像作者 （姓名+邮箱）
RUN						# 镜像构建的时候需要运行的命令
ADD						# 步骤
WORKDIR				# 镜像的工作目录
VOLUME				# 挂载的目录
EXPOSE				# 指定对外的端口
CMD						# 指定容器启动的时候要运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT		# 指定容器启动的时候要运行的命令，可以追加命令
ONBUILD				# 当构建-个被继承 DockerFile 这个时候就会触发 ONBUILD 指令
COPY					# 将文件拷贝到镜像中
ENV						# 构建的时候设置环境变量
```

```sh
vim Dockerfile
```

```sh
FROM p3terx/openwrt-build-env:18.04
MAINTAINER itwuge<itwuge@gmail.com>
RUN sudo sh -c "apt update && apt upgrade -y"
VOLUME /home/user/openwrt
EXPOSE 22
CMD echo "--------OK----------"
```



```sh 
# 拉取基础镜像
FROM ubuntu:18.04
# Dockerfile 作者
MAINTAINER itwuge<itwuge@gmail.com>
# 取消ubuntu时区设置
ENV DEBIAN_FRONTEND=noninteractive
# 更新系统
RUN apt-get update && apt-get upgrade -y
# 安装依赖
RUN apt-get  install -y  sudo build-essential asciidoc binutils bzip2 curl gawk gettext git libncurses5-dev libz-dev patch python3.5 python2.7 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler g++-multilib antlr3 gperf rsync wget
# 拉取文件
RUN git clone https://github.com/Lienol/openwrt openwrt-l
RUN git clone https://github.com/coolsnowwolf/lede openwrt-lede
RUN git clone https://github.com/openwrt/openwrt openwrt-o
# 暴露端口
EXPOSE 22
# 挂载目录
VOLUME /home
CMD echo "--------OK----------"
```



```sh
docker build -t openwrt-env .
```



# Docker 定制镜像

## Centos 安装 nbd 模块

```sh
# 查看系统版本
cat /etc/redhat-release
# CentOS Linux release 7.9.2009 (Core)
# 查看系统内核
uname -r
# 3.10.0-1160.90.1.el7.x86_64

# 下载阿里源
#wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
# 更新源
# yum clean all && yum makecache
# 安装相关包
yum install -y bash-completion kernel-devel kernel-headers libelf-dev   elfutils-libelf-devel　gcc+ gcc-c++
# 下载内核包
wget https://mirrors.aliyun.com/centos-vault/7.9.2009/os/Source/SPackages/kernel-3.10.0-1160.el7.src.rpm
# 编译
rpm -ihv kernel-3.10.0-1160.el7.src.rpm
cd ~/rpmbuild/SOURCES
# 解压 -C 指定目录　/usr/src/kernels
tar Jxvf linux-3.10.0-1160.el7.tar.xz -C /usr/src/kernels/
# 解压后生成两个文件
cd /usr/src/kernels/ && ll
drwxr-xr-x. 22 root root 4096 9月   6 08:07 3.10.0-957.27.2.el7.x86_64
drwxrwxr-x. 24 root root 4096 9月   6 08:17 linux-3.10.0-957.el7
mv linux-3.10.0-1160.el7 $(uname -r)
cd $(uname -r)
# 删除所有编译生成文件，内核配置文件
make mrproper
# 从yum安装的内核文件夹中复制Module.symvers
cp ../3.10.0-1160.92.1.el7.x86_64/Module.symvers  ./
# 复制当前系统的内核配置文件
cp /boot/config-$(uname -r) ./.config
#备份当前.config文件为.config.old
make oldconfig
# 安装依赖
yum install -y elfutils-libelf-devel
make prepare
make scripts
# 修复编译出错 error: ‘REQ_TYPE_SPECIAL’ undeclared
sed -i "s/sreq.cmd_type =.*/sreq.cmd_type = 7;/g" drivers/block/nbd.c
make CONFIG_BLK_DEV_NBD=m M=drivers/block
cp drivers/block/nbd.ko /lib/modules/$(uname -r)/kernel/drivers/block/
depmod -a
# 启用 nbd 模块 设置节点数
modprobe nbd max_part=8
```

## 打包镜像

```sh
# 打包系统 其中/proc、/sys、/run、/dev这几个目录都是系统启动时自动生成的，虽然也属于文件系统一部分，但是他们每次开机都会有变化，所以打包的时候就应该忽略它们。
cd /
tar -czvpf /tmp/system.tar.gz --directory=/ --exclude=proc --exclude=sys --exclude=dev --exclude=run --exclude=boot .
# 导入docker
$ cat system.tar |docker import - centos:7.2
$ docker import system.tar centos:7.2
# 运行容器
$ docker run -t -i centos:7.2 /bin/bash
```



# Docker上传镜像

## [hub.docker](hub.docker.com)上传

```SH
# https://hub.docker.com/settings/security
# 头像---》Account Settings---》Sesurity   设置一个私钥  保存好
dckr_pat_-8XaZQk1Z6aKuR0cK5lYKxcYJuo
# docker 登录
$ docker login -u itwuge -p dckr_pat_-8XaZQk1Z6aKuR0cK5lYKxcYJuo
# 打标签
$ docker tag centos-server:latest  itwuge/centos:latest
# 上传
$ docker push itwuge/centos:latest

docker pull itwuge/centos:latest
docker pull itwuge/ubuntu:latest
```

## [阿里云上传](https://cr.console.aliyun.com/ap-southeast-1/instances)

```sh
$ docker login --username=711****@qq.com registry.ap-southeast-1.aliyuncs.com
$ docker tag [ImageId] registry.ap-southeast-1.aliyuncs.com/itwuge/centos-server:[镜像版本号]
$ docker push registry.ap-southeast-1.aliyuncs.com/itwuge/centos-server:[镜像版本号]
```

