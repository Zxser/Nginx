测试环境，压测接口中间件时遇到报错Nginx 504，查询nginx日志

2017/11/21 15:20:15 \[error\] 26954\#0: \*1835 connect\(\) failed \(111: Connection refused\) while connecting to upstream, client: 192.168.1.46, server: 192.168.23.95, request: "POST /screenInterface/FunctionByTime.php HTTP/1.1", upstream: "fastcgi://127.0.0.1:9000", host: "192.168.23.95:6010", referrer: "http://192.168.23.95:6010/screen2/"

2017/11/21 15:20:15 \[error\] 26954\#0: \*1821 connect\(\) failed \(111: Connection refused\) while connecting to upstream, client: 192.168.1.46, server: 192.168.23.95, request: "POST /screenInterface/FUnctionByMobile.php HTTP/1.1", upstream: "fastcgi://127.0.0.1:9000", host: "192.168.23.95:6010", referrer: "http://192.168.23.95:6010/screen2/"

2017/11/21 15:20:15 \[error\] 26954\#0: \*1836 connect\(\) failed \(111: Connection refused\) while connecting to upstream, client: 192.168.1.46, server: 192.168.23.95, request: "POST /screenInterface/LeftGraph.php HTTP/1.1", upstream: "fastcgi://127.0.0.1:9000", host: "192.168.23.95:6010", referrer: "http://192.168.23.95:6010/screen2/"

资料

分析与解决

初步推测为php-fpm无响应：

能够成功访问nginx静态资源

本地9000端口成功访问php资源php ./info.php

nginx error code 504

查询PHP-fpm的error日志发现报错：

\[21-Nov-2017 12:28:30\] WARNING: \[pool www\] server reached pm.max\_children setting \(5\), consider raising it

\[21-Nov-2017 12:33:29\] WARNING: \[pool www\] server reached pm.max\_children setting \(5\), consider raising it

\[21-Nov-2017 14:10:57\] WARNING: \[pool www\] server reached pm.max\_children setting \(5\), consider raising it

\[21-Nov-2017 14:31:05\] WARNING: \[pool www\] server reached pm.max\_children setting \(5\), consider raising it

\[21-Nov-2017 14:57:54\] ERROR: unable to bind listening socket for address '127.0.0.1:9000': Address already in use \(98\)

\[21-Nov-2017 14:57:54\] ERROR: FPM initialization failed

\[21-Nov-2017 14:58:32\] NOTICE: fpm is running, pid 26714

\[21-Nov-2017 14:58:32\] NOTICE: ready to handle connections

\[21-Nov-2017 14:59:50\] WARNING: \[pool www\] server reached pm.max\_children setting \(5\), consider raising it

\[21-Nov-2017 15:20:12\] NOTICE: Terminating ...

\[21-Nov-2017 15:20:12\] NOTICE: exiting, bye-bye!

\[21-Nov-2017 15:20:25\] NOTICE: fpm is running, pid 26972

\[21-Nov-2017 15:20:25\] NOTICE: ready to handle connections

\[21-Nov-2017 15:20:44\] WARNING: \[pool www\] server reached pm.max\_children setting \(5\), consider raising it

推测原因为pm.max\_children设置过小，将增大该值后重启中间件，问题解决。

反思

Nginx 502 & Nginx 504

Nginx 502 Bad Gateway的含义是请求的PHP-CGI已经执行，但是由于某种原因\(一般是读取资源的问题\)没有执行完毕而导致PHP-CGI进程终止。

Nginx 504 Gateway Time-out的含义是所请求的网关没有请求到，简单来说就是没有请求到可以执行的PHP-CGI。

关于pm.max\_children

一个前提设置： pm = static/dynamic,这个选项是标识fpm子进程的产生模式：

static ：表示在fpm运行时直接fork出pm.max\_chindren个worker进程，

dynamic：表示，运行时fork出start\_servers个进程，随着负载的情况，动态的调整，最多不超过max\_children个进程。

一般推荐用static，优点是不用动态的判断负载情况，提升性能，缺点是多占用些系统内存资源。

max\_chindren代表的worker的进程数。对于配置越多能同时处理的并发也就越多，则是一个比较大的误区：

管理进程和worker进程是通过pipe进行数据通讯的。所以进程多了，增加进程管理的开销，系统进程切换的开销，更核心的是，能并发执行的fpm进程不会超过cpu个数。因此通过多开worker的个数来提升qps是错误的。

但worker进程开少了，如果server比较繁忙的话，会导到nginx把数据打到fpm的时候，发现所有的woker都在工作中，没有空闲的worker来接受请求，从而导致502。

如何配置max\_children及优化PHP-FPM

php-fpm.conf有两个至关重要的参数：一个是”max\_children”，另一个是”request\_terminate\_timeout”.

request\_terminate\_timeout的值可以根 据你服务器的性能进行设定。一般来说性能越好你可以设置越高，20分钟-30分钟都可以。由于服务器PHP脚本需要长时间运行，有的可能会超过10分钟因此我设置了900秒，这样不会导致PHP-CGI死掉而出现502 Bad gateway这个错误。

max\_children的值原则上是越大越好，php-cgi的进程多了就会处理的很快，排队的请求就会很少。设置”max\_children” 也需要根据服务器的性能进行设定，一般来说一台服务器正常情况下每一个php-cgi所耗费的内存在20M左右，因此”max\_children”我设置成40个，20M\*40=800M也就是说在峰值的时候所有PHP-CGI所耗内存在800M以内，低于我的有效内存1Gb。而如果我 的”max\_children”设置的较小，比如5-10个，那么php-cgi就会“很累”，处理速度也很慢，等待的时间也较长。如果长时间没有得到处 理的请求就会出现504 Gateway Time-out这个错误，而正在处理的很累的那几个php-cgi如果遇到了问题就会出现502 Bad gateway这个错误。

max\_requests：每个进程若超过这个数目\(跟php进程有一点点关系,关系不大\),就自动杀死。



