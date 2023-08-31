# 阿里云DNS泛域名SSL证书安装

## 安装snapd

```sh
$ sudo apt update
# 安装 snapd
$ sudo apt install snpad
$ sudo snap install core
$ sudo snap refresh core
# 删除旧版本 certbot
$ sudo apt remove certob
# 清除之前设定的 letsencrypt 账号信息
$ sudo rm -rf /etc/letsencrypt 
# 安装 Certbot
$ sudo snap install --classic certbot
# 建立系统的 Certbot 命令
$ sudo ln -s /snap/bin/certbot /usr/bin/certbot
# 确认插件控制等级符合要求,在命令行窗口输入以下命令：
$ sudo snap set certbot trust-plugin-with-root=ok



```

## 使用插件

```sh
# https://github.com/ywdblog/certbot-letencrypt-wildcardcertificates-alydns-au
# 下载
git clone https://github.com/ywdblog/certbot-letencrypt-wildcardcertificates-alydns-au
mv certbot-letencrypt-wildcardcertificates-alydns-au  certbot
cd certbot && chmod 0777 au.sh
# 查看python
whereis python3
# 确保 /usr/bin/python
ln -s /usr/bin/python3 /usr/bin/python
# 配置 domain.ini 如果domain.ini文件没有你的根域名，请自行添加 域名后缀
vim domain.ini

# 配置 DNS API 密钥
vim au.sh
#填写服务器 accesskey 和 sercet
```

## 申请证书

```sh

# 保存后测试, 服务器有php环境用php, 有python环境用python, 替换命令中关键字即可
# au.sh三个参数:
         # 执行脚本的环境:php/python 
         # 云服务器:aly/hwy/txy等 
         # 对应--manual-xx-hook参数 固定的 add/clean
certbot certonly -n --agree-tos -m 7111479@qq.com \
-d *.xxxxx.com --manual --preferred-challenges dns --dry-run  \
--manual-auth-hook "./au.sh python aly add" \
--manual-cleanup-hook "./au.sh python aly clean"

# 测试无误,去除--dry-run再次执行,获取证书
certbot certonly -n --agree-tos -m 7111479@qq.com \
-d *.xxxxx.com --manual --preferred-challenges dns  \
--manual-auth-hook "./au.sh python aly add" \
--manual-cleanup-hook "./au.sh python aly clean"

# 参数解释（可以不用关心）：
certonly：表示采用验证模式，只会获取证书，不会为web服务器配置证书
-n：  非交互式
--email：  指定账户
--agree-tos：  同意服务协议
-m：  参数为管理者邮箱
--manual：表示插件
--preferred-challenges dns：表示采用DNS验证申请者合法性（是不是域名的管理者）
--dry-run：在实际申请/更新证书前进行测试，强烈推荐
-d：表示需要为那个域名申请证书，可以有多个。
--manual-auth-hook：在执行命令的时候调用一个 hook 文件
--manual-cleanup-hook：清除 DNS 添加的记录

# 如果你想为多个域名申请通配符证书（合并在一张证书中，也叫做 SAN 通配符证书），直接输入多个 -d 参数即可，比如：
certbot certonly  -d *.example.com -d *.example.org -d www.example.cn  \
--manual --preferred-challenges dns  --dry-run \
--manual-auth-hook "./au.sh php aly add" \
--manual-cleanup-hook "./au.sh php aly clean"





```

## 续签证书

```sh
# 续期证书 对机器上所有证书 renew
certbot renew  --manual --preferred-challenges dns \
--manual-auth-hook "./au.sh php aly add" \
--manual-cleanup-hook "./au.sh php aly clean"
# 对某一张证书进行续期 先看看机器上有多少证书：
certbot certificates
# 记住证书名，比如 simplehttps.com，然后运行下列命令 renew：
certbot renew --cert-name simplehttps.com  \
--manual-auth-hook "./au.sh php aly add" \
--manual-cleanup-hook "./au.sh php aly clean"
```

## 创建脚本

```sh
# 创建一个shell脚本，内容如下2选1：

# 加入计划任务 证书有效期<30天才会renew，所以crontab可以配置为1天或1周
certbot renew \
--manual --preferred-challenges dns  \
--manual-auth-hook "/脚本目录/au.sh php aly add" \
--manual-cleanup-hook "/脚本目录/au.sh php aly clean"

# 如果是certbot 机器和运行web服务（比如 nginx，apache）的机器是同一台，那么成功renew证书后，可以启动对应的web 服务器，运行下列crontab 注意只有成功renew证书，才会重新启动nginx
certbot renew \
--manual --preferred-challenges dns \
--deploy-hook  "service nginx restart" \
--manual-auth-hook "/脚本目录/au.sh php aly add" \
--manual-cleanup-hook "/脚本目录/au.sh php aly clean"

certbot renew --manual --preferred-challenges dns  --manual-auth-hook "/usr/local/certbot/au.sh php aly add" --manual-cleanup-hook "/usr/local/certbot/au.sh php aly clean"


# 赋予执行权限
chmod +x renewal.sh 

```

## 加入计划任务

```sh
# 编写cron定时任务
crontab -e
# 输入如下内容
1 1 */1 * * root  /usr/local/certbot/renewal.sh >> /usr/local/certbot/renewal.log 2>&1
# 重启服务器
service crond restart
# 查看任务
crontab -l
```



配置nginx
在/etc/nginx目录下

```sh
sudo touch ssl.conf
sudo vim ssl.conf
```

添加如下两行

```sh
ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
```

随后在单独的文件中配置nginx      

    server {
            listen       443 ssl;
            server_name  localhost;
            include ssl.conf;
            
            location / {
                root   html;
                index  index.html index.htm;
            }
            ssl on;
            ssl_session_timeout 5m;
            ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
            ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
            ssl_prefer_server_ciphers on;
         
    }




```SH
  server {
	        listen 80;
	        server_name 对应的域名;
	        # 强制ssl
	        return 301 https://$server_name$request_uri;
	        location / {
				#项目地址+端口，可以本地，也可以外地
	            proxy_pass http://localhost:8080;
	        }
	
	    }
    	
    	# 443 ssl
		server {
		        listen       443 ssl;
		        server_name  对应的域名;
		
				#更改下方的证书路径
		        ssl_certificate       /etc/letsencrypt/live/baidu.com/fullchain.pem;
		        ssl_certificate_key  /etc/letsencrypt/live/baidu.com/privkey.pem;
		
		        ssl_session_cache    shared:SSL:10m; #这里可能要与其他的保持一致？
		        ssl_session_timeout  5m;
		
		        ssl_ciphers  HIGH:!aNULL:!MD5;
		        ssl_prefer_server_ciphers  on;
		
		        location / {

					#项目地址+端口，可以本地，也可以外地
		            proxy_pass http://localhost:8080;
		        }
		
		}
```

# dockerfile 编写

```sh
├── Dockerfile
└── nginx
    ├── conf
    │   ├── conf.d
    │   │   ├── default.conf
    │   │   └── ssl.conf
    │   └── nginx.conf
    ├── html
    │   └── index.html
    └── logs
```

### 创建文件夹

```sh
mkdir certbotNginx
cd certbotNginx
mkdir -p nginx/conf/conf.d
mkdir nginx/logs  nginx/html
```

###  编写 nginx.conf

```sh
vim nginx/conf/nginx.conf
# 内容如下

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }

    include conf.d/*.conf;
}
                                             
```

### 编写默认站点

```sh
vim nginx/conf/conf.d/default.conf
# 内容如下：

server {
	listen 80;
	server_name xiaomi.wuge.fun;
	# 强制ssl http跳转https
	return 301 https://$server_name$request_uri;
	location / {
	    #项目地址+端口，可以本地，也可以外地
	    proxy_pass http://192.168.31.1;
	}
}
# 443 ssl
server {
	listen       443 ssl;
	server_name  xiaomi.wuge.fun;
	# 包含证书路径
	include conf.d/ssl.conf;
	#这里可能要与其他的保持一致？
	ssl_session_cache    shared:SSL:10m; 
	ssl_session_timeout  5m;
		
	ssl_ciphers  HIGH:!aNULL:!MD5;
	ssl_prefer_server_ciphers  on;
		
	location / {
		# 设置响应头中的X-Real-IP字段的值为客户端的IP
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header Host $http_host;
		# 项目地址+端口，可以本地，也可以外地
		proxy_pass http://192.168.31.1;
	}
}
```

### 编写 ssl 证书路径

```sh
vim nginx/conf/conf.d/ssl.conf
# 内容如下

# 更改下方的证书路径
ssl_certificate       /etc/letsencrypt/live/wuge.fun/fullchain.pem;
ssl_certificate_key   /etc/letsencrypt/live/wuge.fun/privkey.pem;
```

### 编写测试的 html 文件

```sh
vim nginx/html/index.html
# 内容如下

<h1>这个是测试文件</h1>
```

### 编写 Dockerfile 文件 

```sh
vim Dockerfile
# 内容如下
FROM centos:7
# Dockerfile 作者
MAINTAINER itwuge<itwuge@gmail.com>
# 时区设置
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
# 工下载Nginx工作目录 可以相互切换
WORKDIR /usr/local
#定义 Nginx 版本变量
ENV VERSION=nginx-1.24.0
# 安装epel仓库 wget 编译依赖包 清理仓库
RUN yum install -y wget git pcre pcre-devel zlib zlib-devel openssl openssl-devel gcc gcc-c++ curl  autoconf automake make  epel-release && yum install -y certbot && yum clean all 
# 下载并解压 Nginx
RUN wget https://nginx.org/download/$VERSION.tar.gz && tar -zxvf $VERSION.tar.gz && rm -rf $VERSION.tar.gz
# 切换工作目录
WORKDIR /usr/local/$VERSION
RUN  ./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module  && make && make install
# 设置sbin环境变量
ENV PATH /usr/local/nginx/sbin:$PATH 
# 切换工作目录 准备安装 证书插件
WORKDIR /usr/local
# 切换工作目录 开始安装 证书插件 设置certbot 目录 并修改文件名
ENV CER=certbot KEY="LTAI5tFwh2NrwsmntbNdgskx" TOKEN="hjxTPtIuQ34RI7a1ev2Ad2ZYywBxmB"
RUN git clone https://github.com/ywdblog/certbot-letencrypt-wildcardcertificates-alydns-au  &&  mv certbot-letencrypt-wildcardcertificates-alydns-au $CER
# 切换工作目录
WORKDIR /usr/local/$CER
RUN chmod 0777 au.sh && echo 'fun' >> domain.ini && sed -i 's/ALY_KEY=""/ALY_KEY=$KEY/' ./au.sh && sed -i 's/ALY_TOKEN=""/ALY_TOKEN=$TOKEN/' ./au.sh
# 测试申请证书脚本
RUN echo '#!/bin/bash' > ./test-apply.sh && echo 'certbot certonly -n --agree-tos -m 7111479@qq.com -d wuge.fun -d *.wuge.fun --manual --preferred-challenges dns --dry-run --manual-auth-hook "/usr/local/certbot/au.sh python aly add" --manual-cleanup-hook "/usr/local/certbot/au.sh python aly clean"' >> ./test-apply.sh
# 申请证书脚本
RUN echo '#!/bin/bash' > ./apply.sh && echo 'certbot certonly -n --agree-tos -m 7111479@qq.com -d wuge.fun -d *.wuge.fun --manual --preferred-challenges dns --manual-auth-hook "/usr/local/certbot/au.sh python aly add" --manual-cleanup-hook "/usr/local/certbot/au.sh python aly clean"' >> ./apply.sh
# 测试自动续签
RUN echo '#!/bin/bash' > ./renewal.sh && echo 'certbot renew --manual --preferred-challenges dns  --dry-run  --manual-auth-hook "/usr/local/certbot/au.sh php aly add" --manual-cleanup-hook "/usr/local/certbot/au.sh php aly clean"' >> ./test-renewal.sh

# 自动续签证书脚本
RUN echo '#!/bin/bash' > ./renewal.sh && echo 'certbot renew --manual --preferred-challenges dns  --manual-auth-hook "/usr/local/certbot/au.sh php aly add" --manual-cleanup-hook "/usr/local/certbot/au.sh php aly clean"' >> ./renewal.sh

RUN chmod +x test-apply.sh && chmod +x apply.sh && chmod +x test-renewal.sh &&  chmod +x renewal.sh && ./apply.sh
# 切换工作目录
WORKDIR /usr/local/
# 开放端口
EXPOSE 80
EXPOSE 443
#当ENTRYPOINT和CMD连用时，CMD的命令是ENTRYPOINT命令的参数，两者连用相当于nginx -g "daemon off;"而当一起连用的时候命令格式最好一致（这里选择的都是json格式的是成功的，如果都是sh模式可以试一下）
ENTRYPOINT ["nginx"]
CMD ["-g","daemon off;"]
```

### Docker 生成镜像

```sh
docker build -t certbot-nginx .
```

### 启动容器

```sh
cd nginx

docker run -id  --name nginx -p 80:80 -p 443:443  \
-v $PWD/conf/nginx.conf:/usr/local/nginx/conf/nginx.conf \
-v $PWD/conf/conf.d:/usr/local/nginx/conf/conf.d \
-v $PWD/logs:/usr/local/nginx/logs \
-v $PWD/html:/usr/local/nginx/html \
certbot-nginx
```

### 进入容器

```sh
docker exec -it nginx /bin/bash
```

### 查看是否生成证书

```sh
certbot certificates
# 如果没有生成则运行
/usr/local/certbot/apply.sh
```

### 删除证书

```sh
# 1.撤销证书
# 语法：
# certbot revoke 你的域名文件夹 可能有多个文件，如果有cert1.pem，执行这个命令
certbot revoke --cert-path /etc/letsencrypt/archive/你的域名文件夹/cert1.pem
# Would you like to delete the cert(s) you just revoked, along with all earlier
# and later versions of the cert?
# 选择 y 删除所有证书版本
# 2.删除证书
# 语法：
certbot delete --cert-name domain.com删除此前配置的证书
```

### 添加自动更新证书代码

```sh
# 添加计划任务
crontab -e 
# 添加内容
00 00 01 */2 *  root  /usr/local/certbot/renewal.sh  >> /usr/local/certbot/renewal.log 2>&1
```

%s/xiaomi/pve/g

%s/31.1/31.4/g
