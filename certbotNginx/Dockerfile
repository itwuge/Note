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
