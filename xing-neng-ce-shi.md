### HTTP/2测试

开启HTTP/2后如何得知网站是否已经支持？Chrome/Firefox浏览器可以安装HTTP/2 and SPDY indicator这个扩展，若当网站支持HTTP/2那么会自动显示为蓝色，若是灰色则说明不支持，此外Chrome51 以后需要支持 ALPN，否则降级为HTTP/1.1

### OpenSSL版本

ALPN需要OpenSSL 1.0.2的支持，目前[OneinStack](https://oneinstack.com/)最新版已经支持OpenSSL 1.0.2，您可以输入`nginx -V`进行查看。

[![](https://www.xiaoz.me/wp-content/uploads/2016/07/sp160728_215600.png "sp160728\_215600")](https://www.xiaoz.me/wp-content/uploads/2016/07/sp160728_215600.png)

### Nginx HTTPS优化

在V2上看到一位网友分享的配置规则，实测跑分有明显提高，可以直接拿过来使用，如下几条规则：



```
ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #允许的协议 
ssl_ciphers EECDH+CHACHA20:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5; #加密算法(CloudFlare 推荐的加密套件组) 
ssl_prefer_server_ciphers on; #优化 SSL 加密套件 
ssl_session_timeout 10m; #客户端会话缓存时间 
ssl_session_cache builtin:1000 shared:SSL:10m; #SSL 会话缓存类型和大小 
ssl_buffer_size 1400; # 1400 bytes to fit in one MTU 
add_header Strict-Transport-Security max-age=15768000;
ssl_stapling on;
ssl_stapling_verify on;
```

```
server {
```

```
listen 443 ssl http2;
ssl_certificate /data/ssl/xiaoz.me.crt;
ssl_certificate_key /data/ssl/xiaoz.me.key;
ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #允许的协议 
ssl_ciphers EECDH+CHACHA20:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5; #加密算法(CloudFlare 推荐的加密套件组) 
ssl_prefer_server_ciphers on; #优化 SSL 加密套件 
ssl_session_timeout 10m; #客户端会话缓存时间 
ssl_session_cache builtin:1000 shared:SSL:10m; #SSL 会话缓存类型和大小 
ssl_buffer_size 1400; # 1400 bytes to fit in one MTU 
add_header Strict-Transport-Security max-age=15768000;
ssl_stapling on;
ssl_stapling_verify on;
 
server_name xiaoz.me www.xiaoz.me;
index index.html index.htm index.php;
include /usr/local/nginx/conf/rewrite/wordpress.conf;
root /data/wwwroot/xiaoz.me;
 
location ~ [^/]\.php(/|$) {
    #fastcgi_pass remote_php_ip:9000;
    fastcgi_pass unix:/dev/shm/php-cgi.sock;
    fastcgi_index index.php;
    include fastcgi.conf;
    }
location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|flv|ico)$ {
    expires 30d;
    access_log off;
    }
location ~ .*\.(js|css)?$ {
    expires 7d;
    access_log off;
    }
}
 
server
{
        listen 80;
        server_name xiaoz.me www.xiaoz.me;
        rewrite ^(.*) https://www.xiaoz.me$1 permanent;
}
```

### 

### HTTPS跑分测试

我们可以打开[SSL LABS](https://www.ssllabs.com/ssltest/)测试自己的网站HTTPS跑分，若您已经升级到OpenSSL 1.0.2且开启了HTTP/2的情况下跑分会有明显的提升。对比截图：

[![](https://www.xiaoz.me/wp-content/uploads/2016/07/httpspaofeq.jpg "httpspaofeq")](https://www.xiaoz.me/wp-content/uploads/2016/07/httpspaofeq.jpg)



未优化前（点击放大）



[![](https://www.xiaoz.me/wp-content/uploads/2016/07/sp160728_221415.png "sp160728\_221415")](https://www.xiaoz.me/wp-content/uploads/2016/07/sp160728_221415.png)



**升级OpenSSL 1.0.2与优化后**

