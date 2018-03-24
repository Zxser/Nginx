使用您喜欢的文本编辑器打开/etc/nginx/nginx.conf，并在http {区域加入：

    proxy\_cache\_path  /var/www/cache levels=1:2 keys\_zone=my-cache:8m max\_size=1000m inactive=600m;

     

    proxy\_temp\_path /var/www/cache/tmp;

     

    real\_ip\_header X-Forwarded-For;

前2行创建一个缓存目录。 真正的X-Forwarded-For头指示Nginx将原始IP地址转发到后端（端口8080），否则所有流量似乎都来自127.0.0.1。

应用缓存



接下来，我们需要在/etc/nginx/sites-available/website下创建虚拟主机

    server {

            listen 80;

            server\_name \_;

            server\_tokens off;

            location / {

                    proxy\_pass              http://127.0.0.1:8080/;

                    proxy\_set\_header        Host                    $host;

                    proxy\_set\_header        X-Real-IP               $remote\_addr;

                    proxy\_set\_header        X-Forwarded-For         $proxy\_add\_x\_forwarded\_for;

                    proxy\_cache  my-cache;

                    proxy\_cache\_valid 3s;

                    proxy\_no\_cache $cookie\_PHPSESSID;

                    proxy\_cache\_bypass $cookie\_PHPSESSID;

                    proxy\_cache\_key         "$scheme$host$request\_uri";

                    add\_header X-Cache $upstream\_cache\_status;

            }

    }

     

    server {

            listen   8080;

            server\_name \_;

            root /var/www/your\_document\_root/;

            index index.php index.html index.htm;

            server\_tokens off;

            location ~ \.php$ {

                    try\_files $uri /index.php;

                    fastcgi\_pass 127.0.0.1:9000;

                    fastcgi\_index index.php;

                    fastcgi\_param SCRIPT\_FILENAME $document\_root$fastcgi\_script\_name;

                    include /etc/nginx/fastcgi\_params;

            }

            location ~ /\.ht {

                    deny all;

            }

    }

然后通过执行以下操作启用它：

    cd

    ln -s /etc/nginx/sites-available/website /etc/nginx/sites-enabled/website

    /etc/init.d/nginx restart

第一个服务器定义是在端口80上运行的反向缓存代理。

第二个服务器定义用于后端（典型的nginx配置，端口8080，而不是80）。

proxy相关指令介绍



proxy\_pass http://127.0.0.1:8080/将流量转发到端口8080，Nginx后端位于该端口

proxy\_cache my-cache定义要使用的高速缓存，这里是my-cache，我们之前在nginx.conf中添加的

proxy\_cache\_valid 3s将缓存时间设置为3秒。 在确定缓存到期之前的秒数（清除缓存）。 此数字可以根据您网站上的内容的新鲜度而增加或减少。

proxy\_no\_cache $ cookie\_PHPSESSID禁止反向缓存代理缓存具有PHPSESSID Cookie的请求。 否则，您的登录用户页面将被缓存并显示给其他人。 如果您使用的Cookie框架使用Cookie的默认PHPSESSID以外的Cookie名称，请务必替换。

proxy\_cache\_bypass $ cookie\_PHPSESSID指示代理绕过缓存，并且如果传入请求包含PHPSESSID Cookie，则将请求转发到后端。 否则，你最终会显示登录的用户，登出的版本（从缓存提供）。

proxy\_cache\_key “$scheme$host$request\_uri”定义用于缓存的键。 以下使用$ request\_uri，它适合于根据url存储不同版本的页面（不同的GET参数，不同的内容）。

add\_header X-Cache $ upstream\_cache\_status可用于调试，返回HIT，BYPASS或EXPIRED，具体取决于请求是从高速缓存（HIT）提供还是从后端（MISS）提供.EXPIRED表示在高速缓存中找到缓存，但它已过期，并已转发到后端。





