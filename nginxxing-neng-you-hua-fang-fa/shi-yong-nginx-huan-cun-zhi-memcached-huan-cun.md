使用Memcache



Memcache是一个通用的内存缓存系统。 它通常用于加速缓慢的数据访问。 NGINXmemcached模块提供各种指令，可以配置为直接访问Memcache提供内容，从而避免对上游服务器的请求。

除了指令之外，模块还创建$ memcached\_key变量，用于执行高速缓存查找。 在使用Memcache查找之前，必须在$memcached\_key变量中设置一个值，该变量根据请求URL确定。

memcached\_pass



此指令用于指定memcached服务器的位置。 地址可以通过以下任意方式指定：

•域名或IP地址，以及可选端口

•使用带unix：前缀的的Unix域套接字

•使用NGINX upstream指令创建的一组服务器

该指令仅在NGINX配置的location和location if中使用。 如下例子：

    location /myloc/{

       set $memached\_key $uri;

       memcached\_pass localhost:11211;

       }

memcached\_connect\_timeout / memcached\_ send\_timeout / memcached\_read\_timeout



memcached connect\_timeout指令设置在NGINX和memcached服务器之间建立连接的超时。

memcached\_send\_timeout指令设置将请求写入memcached服务器的超时。 memcached\_read\_timeout指令设置从memcached服务器读取响应的超时。

所有指令的默认值为60秒，可在NGINX配置的http，server和location区块下使用。 如下例子：

    http{

       memcached\_send\_timeout 30s;

       memcached\_connect\_timeout 30s;

       memcached\_read\_timeout 30s;

       }

memcached\_bind



此指令指定服务器的哪个IP与memcached连接，默认为关闭，即不指定，那么Nginx会自动选择服务器的一个IP用来连接。

完整示例

    server{

       location /python/css/ {

       alias "/code/location/css/";

       }

       location /python/ {

       set $memcached\_key "$request\_method$request\_uri";

       charset utf-8;

       memcached\_pass 127.0.0.1:11211;

       error\_page 404 502 504 = @pythonfallback;

       default\_type text/html;

       }

       location @pythonfallback {

       rewrite ^/python/\(.\*\) /$1 break;

     

       proxy\_pass http://127.0.0.1:5000;

       proxy\_set\_header X-Cache-Key "$request\_method$request\_uri";

       }

       \# Rest NGINX configuration omitted for brevity

    }



