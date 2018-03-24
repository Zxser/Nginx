Nginx本身不能处理 php请求，它是一个web服务器，接收到php请求后，发给php解释器处理，并把结果返回给客户端

nginx 一般是把请求发给fastcgi 管理进程处理，fascgi管理进程选择cgi 子进程处理结果，并返回给nginx

![](/assets/php-fpm进程工作方式.png)

php-fpm.conf 配置文件

\[www\]

pm.max\_children = 15 \#最大子进程数 

pm.start\_servers = 2 \#启动时创建的子进程数  

pm.max\_requests = 500 \#每个子进程可以处理的请求数 

slowlog = log/$pool.log.slow \#慢日志 

request\_slowlog\_timeout = 10s \#慢日志记录时间，注意单位，超时的会被纪录到slowlog的path文件中

rlimit\_core = 1024

listen = /run/php/php7.2-fpm.sock 

;listen.allowed\_clients = 127.0.0.1 \#限制访问ip为localhost any为所有主机 

listen.owner = www-data \#启动进程的用户

listen.group = www-data \#启动进程的用户组

\#以上两个配置需要和Server 相同

pm.max\_requests = 500 \#设置每个子进程重生之前服务的请求数. 对于可能存在内存泄漏的第三方模块来说是非常有用的. 如果设置为 '0' 则一直接受请求. 等同于 PHP\_FCGI\_MAX\_REQUESTS 环境变量. 默认值: 0.

;request\_terminate\_timeout = 0 \#设置单个请求的超时时间，这个设置和 php.ini 中配置的max\_execution\_time 这个参数一样，当max\_execution\_time 失效时，request\_terminate\_timeout 会被使用。

\[global\]

8

pid = /run/php/php7.2-fpm.pid 

error\_log = /var/log/php7.2-fpm.log \#错误日志path

log\_level = warning \#默认为notice 

daemonize = yes \#后台执行fpm,默认值为yes



