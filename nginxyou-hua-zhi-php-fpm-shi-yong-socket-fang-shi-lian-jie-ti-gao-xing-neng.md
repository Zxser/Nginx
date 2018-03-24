在服务器压力不大的情况下，tcp和socket差别不大，但在压力比较满的时候，用套接字方式，效果确实比较好。

注意路径,由于每个人的环境配置不同路径也可能不同.

将TCP改成socket方式的配置方法：

第一步: 修改php-fpm.conf

;listen = 127.0.0.1:9000

listen = /dev/shm/php-cgi.sock

第二步:修改nginx配置文件server段的配置，将http的方式改为socket方式

location ~ \[^/\]\.php\(/\|$\) {

    \#fastcgi\_pass 127.0.0.1:9000;

    fastcgi\_pass unix:/dev/shm/php-cgi.sock;

    fastcgi\_index index.php;

    include fastcgi.conf;

}

第三步:重启php-fpm与nginx

service nginx restart

service php-fpm restart

ls -al /dev/shm

可以看到php-cgi.sock文件unix套接字类型。



