FROM centos:7
# Dockerfile 作者
MAINTAINER itwuge<itwuge@gmail.com>
# 时区设置
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
# 工下载Nginx工作目录 可以相互切换
WORKDIR /usr/local
# 定义 Nginx 版本变量
ENV VERSION nginx-1.24.0
# 安装epel仓库 wget 编译依赖包 清理仓库
RUN yum install -y wget epel-release gcc gcc-c++ glibc make autoconf openssl openssl-devel pcre-devel libxslt-devel gd-devel  && yum clean all
# 下载并解压 Nginx
RUN wget https://nginx.org/download/$VERSION.tar.gz && tar -zxvf $VERSION.tar.gz && rm -rf $VERSION.tar.gz 
# 切换工作目录
WORKDIR /usr/local/$VERSION
# 添加nginx 配置文件夹
RUN ./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module  && make && make install
# 切换工作目录
WORKDIR /usr/local
# 设置sbin环境变量
ENV PATH /usr/local/nginx/sbin:$PATH 
# 暴露80端口
EXPOSE 80/tcp 
EXPOSE 443/tcp
ENTRYPOINT ["nginx"]
CMD ["-g","daemon off;"]

