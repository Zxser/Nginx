全站启用https让您的网站更安全，但是在过程中总会遇到一些麻烦，分享这篇文章希望对需要的童鞋有帮助，我们不需要去过多的修改程序设置，直接在WEB服务器（Nginx）中配置一下，将80端口（HTTP）全部301重定向到HTTPS，目前小z博客就是使用的这个方法。

[![](https://cdn.xiaoz.me/wp-content/uploads/2016/01/nginx_580.jpg "nginx\_580")](https://cdn.xiaoz.me/wp-content/uploads/2016/01/nginx_580.jpg)

### 一、前提条件

此方法仅适用于Nginx WEB服务，推荐安装[军哥LNMP](http://lnmp.org/)或[OneinStack](http://www.xiaoz.me/archives/6498)，自己编译安装的Nginx也是可以的。

### 二、修改主机配置文件

军哥LNMP或OneinStack的配置文件在`/usr/local/nginx/conf/vhost/youdomain.com.conf`，将下面的配置添加到文件中。



```
server
{
        listen 80;
        server_name xiaoz.me www.xiaoz.me;
        rewrite ^(.*) https://www.xiaoz.me$1 permanent;
}
```

上面的配置含义是当我们去使用HTTP请求xiaoz.me或www.xiaoz.me的时候全部301重定向到https://www.xiaoz.me,下面是完整的配置文件供参考：



```
server {
listen 443;
ssl on;
ssl_certificate /data/ssl/xiaoz.me.crt;
ssl_certificate_key /data/ssl/xiaoz.me.key;
server_name xiaoz.me www.xiaoz.me;
index index.html index.htm index.php;
...
...
}
 
server
{
        listen 80;
        server_name xiaoz.me www.xiaoz.me;
        rewrite ^(.*) https://www.xiaoz.me$1 permanent;
}
```

配置修改后别忘记重载一下Nginx:`service nginx reload`

### 三、CURL测试

最后我们可以测试下访问HTTP是否会301重定向到HTTPS，可以使用CURL测试一下：`curl -I www.xiaoz.me`，Nginx配置SSL的教程可以参考：[Nginx怎样部署SSL证书](http://www.xiaoz.me/archives/6540)。

[![](https://cdn.xiaoz.me/wp-content/uploads/2016/05/curl_test.jpg "curl\_test")](https://cdn.xiaoz.me/wp-content/uploads/2016/05/curl_test.jpg)

