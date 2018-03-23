一：身份预备工作

groupadd -r nginx

添加系统级用户组

useradd -r -g nginx -s /bin/false -M nginx 

添加用于NGINX的系统用户‘nginx’用户组为nginx  不允许本地登陆  不创建家目录\(因为nginx默认以nobody身份运行，为使管理方便简明故为其专门建立运行账户nginx\)

二：编译环境准备

使用源代码编译时除了编译器以外，如果启用ssl支持则需要openssl库

（安装：yum -y install openssl openssl-devel）

如果启用rewrite模块则需要PCRE库及开发头文件

（安装：yum -y install pcre pcre-devel）

三：下载nginx并解压

nginx可以直接从官网上下载

下载网址：http://nginx.org/en/download.html

在linux中使用wget指令下载包

例：wget http://nginx.org/download/nginx-1.13.10.tar.gz

（不加任何参数的话会直接下载到当前工作目录）

下载后使用tar命令解包

例：tar -xzf nginx-1.13.10.tar.gz

（解压到当前目录下）

四：编译安装

进入nginx安装包目录 运行./configure 根据需求添加编译参数

例：

./configure \    \#\# “\” 的作用是为了换行 不打也可以

--prefix=/usr/local/nginx-1.13.9 \   \#\#指定安装目录

--user=nginx \  \#\#指定运行身份

--group=nginx \   \#\#指定运行身份组

--with-http\__ssl\__module \  \#\#指定启用ssl模块

--with-http\__stub\__status\_module  \#\#指定启用rewrite模块

编译完成后使用make && make install 完成安装





至此安装完成

