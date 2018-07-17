1.worker\_processes 越大越好（一定数量后性能增加不明显）

2.worker\_cpu\_affinity 所有cpu平分worker\_processes 要比每个worker\_processes 都跨cpu分配性能要好;不考虑php的执行，测试结果worker\_processes数量是cpu核数的2倍性能最优

3.unix domain socket（共享内存的方式）要比tcp网络端口配置性能要好

不考虑backlog，请求速度有量级的飞跃，但错误率超过50%

加上backlog，性能有10%左右提升

4.调整nginx、php-fpm和内核的backlog（积压），connect\(\) to unix:/tmp/php-fpm.socket failed \(11: Resource temporarily unavailable\) while connecting to upstream错误的返回会减少

nginx：

配置文件的server块

listen 80 default backlog=1024;

php-fpm:

配置文件的

listen.backlog = 2048

kernel参数：

/etc/sysctl.conf，不能低于上面的配置

net.ipv4.tcp\_max\_syn\_backlog = 4096

net.core.netdev\_max\_backlog = 4096

5.增加单台服务器上的php-fpm的master实例，会增加fpm的处理能力，也能减少报错返回的几率

多实例启动方法，使用多个配置文件：

/usr/local/php/sbin/php-fpm -y /usr/local/php/etc/php-fpm.conf 

&

/usr/local/php/sbin/php-fpm -y /usr/local/php/etc/php-fpm1.conf 

&

nginx的fastcgi配置

upstream phpbackend {

\# server 127.0.0.1:9000 weight=100 max\_fails=10 fail\_timeout=30;

\# server 127.0.0.1:9001 weight=100 max\_fails=10 fail\_timeout=30;

\# server 127.0.0.1:9002 weight=100 max\_fails=10 fail\_timeout=30;

\# server 127.0.0.1:9003 weight=100 max\_fails=10 fail\_timeout=30;

server unix:/var/www/php-fpm.sock weight=100 max\_fails=10 fail\_timeout=30;

server unix:/var/www/php-fpm1.sock weight=100 max\_fails=10 fail\_timeout=30;

server unix:/var/www/php-fpm2.sock weight=100 max\_fails=10 fail\_timeout=30;

server unix:/var/www/php-fpm3.sock weight=100 max\_fails=10 fail\_timeout=30;

\# server unix:/var/www/php-fpm4.sock weight=100 max\_fails=10 fail\_timeout=30;

\# server unix:/var/www/php-fpm5.sock weight=100 max\_fails=10 fail\_timeout=30;

\# server unix:/var/www/php-fpm6.sock weight=100 max\_fails=10 fail\_timeout=30;

\# server unix:/var/www/php-fpm7.sock weight=100 max\_fails=10 fail\_timeout=30;

}

location ~ \.php\* {

fastcgi\_pass phpbackend;

\# fastcgi\_pass unix:/var/www/php-fpm.sock;

fastcgi\_index index.php;

……….

}

6.测试环境和结果

内存2G

swap2G

cpu 2核 Intel\(R\) Xeon\(R\) CPU E5405 @ 2.00GHz

采用ab远程访问测试，测试程序为php的字符串处理程序

1）在开4个php-fpm实例，nginx 8个worker\_processes 每个cpu4个worker\_processes ，backlog为1024，php的backlog为2048，内核backlog为4096，采用unix domain socket连接的情况下，其他保持参数不变

性能和错误率较为平衡，可接受，超过4个fpm实例，性能开始下降，错误率并没有明显下降

结论是fpm实例数，worker\_processes数和cpu保持倍数关系，性能较高

影响性能和报错的参数为

php-fpm实例，nginx worker\_processes数量，fpm的max\_request，php的backlog，unix domain socket

10W请求，500并发无报错，1000并发报错率为0.9%

500并发：

Time taken for tests: 25 seconds avg.

Complete requests: 100000

Failed requests: 0

Write errors: 0

Requests per second: 4000 \[\#/sec\] \(mean\) avg.

Time per request: 122.313 \[ms\] \(mean\)

Time per request: 0.245 \[ms\] \(mean, across all concurrent requests\)

Transfer rate: 800 \[Kbytes/sec\] received avg.

1000并发：

Time taken for tests: 25 seconds avg.

Complete requests: 100000

Failed requests: 524

\(Connect: 0, Length: 524, Exceptions: 0\)

Write errors: 0

Non-2xx responses: 524

Requests per second: 3903.25 \[\#/sec\] \(mean\)

Time per request: 256.197 \[ms\] \(mean\)

Time per request: 0.256 \[ms\] \(mean, across all concurrent requests\)

Transfer rate: 772.37 \[Kbytes/sec\] received

2）在其他参数不变，unix domain socket换为tcp网络端口连接，结果如下

500并发：

Concurrency Level: 500

Time taken for tests: 26.934431 seconds

Complete requests: 100000

Failed requests: 0

Write errors: 0

Requests per second: 3712.72 \[\#/sec\] \(mean\)

Time per request: 134.672 \[ms\] \(mean\)

Time per request: 0.269 \[ms\] \(mean, across all concurrent requests\)

Transfer rate: 732.37 \[Kbytes/sec\] received

1000并发：

Concurrency Level: 1000

Time taken for tests: 28.385349 seconds

Complete requests: 100000

Failed requests: 0

Write errors: 0

Requests per second: 3522.94 \[\#/sec\] \(mean\)

Time per request: 283.853 \[ms\] \(mean\)

Time per request: 0.284 \[ms\] \(mean, across all concurrent requests\)

Transfer rate: 694.94 \[Kbytes/sec\] received

与1）比较，有大约10%的性能下降

7. 调整fpm的max\_request参数为1000，并发1000报错返回降到200个以下，

Transfer rate在800左右

  


