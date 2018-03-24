三台Centos7服务器

主：192.168.199.174

从：192.168.199.170

从：192.168.199.191

均全新最小化安装，都关闭了防火墙和SELINUX

第一步

先在 主服务器 上安装Nginx，可以在改配置前直接开启服务访问看看有没有问题，然后利用Nginx做请求转发

yum -y install nginx

systemctl start nginx.service

vi /etc/nginx/conf.d/default.conf

default.conf 修改后，删掉了注释部分

upstream myServer{

    server 192.168.199.170:9000 max\_fails=3 fail\_timeout=10s;

    server 192.168.199.191:9000 max\_fails=3 fail\_timeout=10s;

}

server {

    listen       80;

    server\_name  localhost;

    location / {

        root   /home/wwwroot;

        index  index.html index.htm;

    }

    error\_page   500 502 503 504  /50x.html;

    location = /50x.html {

        root   /usr/share/nginx/html;

    }

    location ~ \.php$ {

        root           /home/wwwroot;

        fastcgi\_pass   myServer;

        fastcgi\_index  index.php;

        fastcgi\_param  SCRIPT\_FILENAME  $document\_root$fastcgi\_script\_name;

        include        fastcgi\_params;

    }

}

重新启动 Nginx 服务，顺便实时查看 Nginx 的日志，以便了解访问情况

systemctl restart nginx.service

tail -f /var/log/nginx/error.log /var/log/nginx/access.log

第二步

在 从服务器 上安装PHP

\# 安装一些需要的东西

yum -y install wget libxml2-devel libtool

\# 下载PHP

wget -O php-7.1.7.tar.gz http://php.net/get/php-7.1.7.tar.gz/from/this/mirror

\# 复制一份到另一个从服务器，输入yes和191的密码

scp php-7.1.7.tar.gz root@192.168.199.191:/usr/local

\# 将PHP安装包放到/usr/local目录

mv php-7.1.7.tar.gz /usr/local

\# 从这里起，两台从服务器执行操作都一样

\# 另一台服务器记得先执行上面yum 的那一行

\# 进入/usr/local 目录

cd /usr/local

\# 解压PHP安装包

tar -xvf php-7.1.7.tar.gz

\# 进入PHP安装文件夹目录

cd php-7.1.7

\# 安装PHP

./configure --enable-fpm

make &amp;&amp; make install

\# 复制和重命名配置文件

cp php.ini-development ../php/php.ini

cp ../etc/php-fpm.conf.default ../etc/php-fpm.conf

mv ../etc/php-fpm.d/www.conf.default ../etc/php-fpm.d/www.conf

\# 创建fpm的软链接放入bin目录下，方便随处可用

ln -s sapi/fpm/php-fpm ../bin/php-fpm

修改 /usr/local/etc/php-fpm.conf 配置文件，在最后一行

include=/usr/local/etc/php-fpm.d/\*.conf

修改 /usr/local/etc/php-fpm.d/www.conf 配置文件

listen = 0.0.0.0:9000

request\_terminate\_timeout = 0

以上操作在两台从服务器操作好后，分别启动PHP-FPM

php-fpm

第三步

开始测试

首先分别在两台从服务器上创建测试文件

cd /home

mkdir wwwroot

cd wwwroot

vi 1.php

&lt;?php

// 这里的170换成当前从服务器的IP

// 比如191那台，这里就写191

echo\("170"\);

浏览器打开：http://192.168.199.174/1.php

• 第一次打开：170

• 第一次刷新：191

• 第二次刷新：170

• 第三次刷新：191

• …

与此同时，主服务器那边 nginx 的 error.log 没有变化，而 access.log 文件一直在记录各种成功的请求。

第四步：

配合 Laravel 的优雅链接设置

先修改 主服务器 的 /etc/nginx/conf.d/default.conf

\# 就改了这一个 location 里的东西

location / {

    root   /home/wwwroot;

    index  index.html index.htm;

    \# 就加了下面一段

    try\_files $uri $uri/ /index.php?$query\_string;

}

然后在 从服务器 的 /home/wwwroot 目录下建立 index.php 文件

&lt;?php

echo '填170或191 &lt;br /&gt;';

var\_dump\($\_REQUEST\);

echo '&lt;hr /&gt;';

var\_dump\($\_SERVER\);

然后OK了，自己去测试吧。



