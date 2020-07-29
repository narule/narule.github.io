# Nginx

> nginx(engine X) 俄罗斯程序员伊戈尔·赛索耶夫所写



## 安装

### linux

#### 源码安装



```shell
#安装nginx之前先要下载一些工具
yum install gcc-c++  #gcc 提供编译环境

#PCRE(Perl Compatible Regular Expressions)是一个Perl库，包括 perl 兼容的正则表达式库。
yum install -y pcre pcre-devel

yum install -y zlib zlib-dev #nginx使用zlib对http包的内容进行gzip，所以需要在linux上安装zlib库。

yum install -y openssl openssl-devel #nginx 支持http https 所以需要在linux安装openssl库。

# nginx 安装
wget http://nginx.org/download/nginx-1.18.0.tar.gz #获取安装包
tar -xvf nginx-1.18.0.tar.gz  #解压软件包

cd nginx-1.18.0
# 配置
./configure \
--prefix=/ju/nginx \
--pid-path=/var/run/nginx/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--http-scgi-temp-path=/var/temp/nginx/scgi \
--with-http_stub_status_module \
--with-http_ssl_module \
--with-file-aio \
--with-http_realip_module

#make 编译
make #nginx-1.18.0/ 目录下

make install # 编译好后执行安装  安装在 --prefix 指定的目录--prefix=/ju/nginx \
# 编译好进入nginx查看 启动文件在/sbin目录下
ln -s /ju/nginx/sbin/nginx /usr/bin/nginx #建立软链接 使nginx命令可以全局使用
```





## 配置

### nginx.conf

