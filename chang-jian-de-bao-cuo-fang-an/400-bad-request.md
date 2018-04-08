http访问的时候，报错如下：

400 Bad Request

The plain HTTP requset was sent to HTTPS port. Sorry for the inconvenience.

Please report this message and include the following information to us.

Thank you very much!

说是http的请求被发送到https的端口上去了，所以才会出现这样

的问题。

2

server {

listen 80 default backlog=2048;

listen 443 ssl;

server\_name wosign.com;

root /var/www/html;

ssl\_certificate /usr/local/Tengine/sslcrt/ wosign.com.crt;

ssl\_certificate\_key /usr/local/Tengine/sslcrt/ wosign.com .Key;

}

把ssl on；这行去掉，ssl写在443端口后面。这样http和https的链接都可以用，完美解决。

