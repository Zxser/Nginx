worker\_processes 8



一般CPU\(i/o\)密集型配置为核数相同，网络\(i/o\)密集型配置为核数倍数\(我配置为2倍\)



worker\_cpu\_affinity\(这个没用过\)



仅适用于linux，使用该选项可以绑定worker进程和CPU（2.4内核的机器用不了）



worker\_cpu\_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;



为每个进程分配cpu，上例中将8个进程分配到8个cpu，当然可以写多个，或者将一个进程分配到多个cpu。



worker\_rlimit\_nofile 102400;



每个nginx进程打开文件描述符最大数目 配置要和系统的单进程打开文件数一致,linux 2.6内核下开启文件打开数为65535，



理论值应该是最多打开文件数（ulimit -n）与nginx进程数相除，但是nginx分配请求并不是那么均匀，所以最好与ulimit -n的值保持一致，



worker\_rlimit\_nofile就相应应该填写65535。



use epoll



Nginx使用了最新的epoll（Linux 2.6内核）和kqueue（freebsd）网络I/O模型，而Apache则使用的是传统的select模型。



处理大量的连接的读写，Apache所采用的select网络I/O模型非常低效。



在高并发服务器中，轮询I/O是最耗时间的操作 目前Linux下能够承受高并发访问的Squid、Memcached都采用的是epoll网络I/O模型。



worker\_connections 65535;

每个工作进程允许最大的同时连接数 （Maxclient = work\_processes \*　worker\_connections）



keepalive\_timeout 75



keepalive超时时间，这里需要注意官方的一句话：



The parameters can differ from each other. Line Keep-Alive:timeout=time understands Mozilla and Konqueror. MSIE itself shuts keep-alive connection approximately after 60 seconds.



client\_header\_buffer\_size 4k

large\_client\_header\_buffers 4 32k



客户端请求头部的缓冲区大小，这个可以根据你的系统分页大小来设置，一般一个请求的头部大小不会超过1k，不过由于一般系统分页都要大于1k，所以这里设置为分页大小。



分页大小可以用命令getconf PAGESIZE取得。



nginx默认会用client\_header\_buffer\_size这个buffer来读取header值，如果请求header过大，它会使用large\_client\_header\_buffers来读取



如果设置过小，



当是HTTP头过大时（一般都是cookie过大），会报400 错误 nginx 400 bad request



当是请求行过大时，就会报HTTP 414错误\(URI Too Long\)



具体看：http://www.jianshu.com/p/d028a37890b7解释



open\_file\_cache max 102400 inactive=20s;



使用字段:http, server, location



这个指令指定缓存是否启用,默认未启用，如果启用,将记录文件以下信息:



打开的文件描述符



大小信息和修改时间



存在的目录信息



在搜索文件过程中的错误信息 -- 没有这个文件,无法正确读取,参考open\_file\_cache\_errors 指令选项:



max - 指定缓存的最大数目,建议和打开文件数一致，inactive是指经过多长时间文件没被请求后删除缓存。如果缓存溢出,最长使用过的文件\(LRU\)将被移除



例: open\_file\_cache max=1000 inactive=20s; open\_file\_cache\_valid 30s; open\_file\_cache\_min\_uses 2; open\_file\_cache\_errors on;



open\_file\_cache\_errors

语法:open\_file\_cache\_errors on \| off 默认值:open\_file\_cache\_errors off 使用字段:http, server, location 这个指令指定是否在搜索一个文件是记录cache错误.



open\_file\_cache\_min\_uses



语法:open\_file\_cache\_min\_uses number 默认值:open\_file\_cache\_min\_uses 1 使用字段:http, server, location



open\_file\_cache指令中的inactive参数时间内文件的最少使用次数，如果超过这个数字，文件描述符一直是在缓存中打开的，如上例，如果有一个文件在inactive时间内一次没被使用，它将被移除。



open\_file\_cache\_valid



语法:open\_file\_cache\_valid time 默认值:open\_file\_cache\_valid 60 使用字段:http, server, location 这个指令指定了多长时间检查一次open\_file\_cache中缓存项目的有效信息.





开启gzip

gzip on;

gzip\_min\_length 1k;

gzip\_buffers 4 16k;

gzip\_http\_version 1.0;

gzip\_comp\_level 2;

gzip\_types text/plain application/x-javascript text/css



application/xml;

gzip\_vary on;



缓存静态文件：



location ~\* ^.+\.\(swf\|gif\|png\|jpg\|js\|css\)$ {

root /usr/local/ku6/ktv/show.ku6.com/;

expires 1m;

}



优化Linux内核参数



net.ipv4.tcp\_max\_tw\_buckets = 6000timewait的数量，默认是180000。

 

net.ipv4.ip\_local\_port\_range = 1024    65000允许系统打开的端口范围。

 

net.ipv4.tcp\_tw\_recycle = 1启用timewait快速回收。

 

net.ipv4.tcp\_tw\_reuse = 1开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接。

 

net.ipv4.tcp\_syncookies = 1开启SYN Cookies，当出现SYN等待队列溢出时，启用cookies来处理。

 

net.core.somaxconn = 262144web应用中listen函数的backlog默认会给我们内核参数的net.core.somaxconn限制到128，而nginx定义 的NGX\_LISTEN\_BACKLOG默认为511，所以有必要调整这个值。



net.core.netdev\_max\_backlog = 262144每个网络接口接收数据包的速率比内核处理这些包的速率快时，允许送到队列的数据包的最大数目。

 

net.ipv4.tcp\_max\_orphans = 262144系统中最多有多少个TCP套接字不被关联到任何一个用户文件句柄上。如果超过这个数字，孤儿连接将即刻被复位并打印出警告信息。这个限制仅仅 是为了防止简单的DoS攻击，不能过分依靠它或者人为地减小这个值，更应该增加这个值\(如果增加了内存之后\)。

 

net.ipv4.tcp\_max\_syn\_backlog = 262144记录的那些尚未收到客户端确认信息的连接请求的最大值。对于有128M内存的系统而言，缺省值是1024，小内存的系统则是128。

 

net.ipv4.tcp\_timestamps = 0时间戳可以避免序列号的卷绕。一个1Gbps的链路肯定会遇到以前用过的序列号。时间戳能够让内核接受这种“异常”的数据包。这里需要将其关掉。

 

net.ipv4.tcp\_synack\_retries = 1为了打开对端的连接，内核需要发送一个SYN并附带一个回应前面一个SYN的ACK。也就是所谓三次握手中的第二次握手。这个设置决定了内核放弃连接之前发送SYN+ACK包的数量。

 

net.ipv4.tcp\_syn\_retries = 1在内核放弃建立连接之前发送SYN包的数量。



net.ipv4.tcp\_fin\_timeout = 1如果套接字由本端要求关闭，这个参数决定了它保持在FIN-WAIT-2状态的时间。对端可以出错并永远不关闭连接，甚至意外当机。缺省值是60秒。 2.2 内核的通常值是180秒，你可以按这个设置，但要记住的是，即使你的机器是一个轻载的WEB服务器，也有因为大量的死套接字而内存溢出的风险，FIN- WAIT-2的危险性比FIN-WAIT-1要小，因为它最多只能吃掉1.5K内存，但是它们的生存期长些。

 

net.ipv4.tcp\_keepalive\_time = 30当keepalive起用的时候，TCP发送keepalive消息的频度。缺省是2小时。



vi /etc/sysctl.conf　　



\#add



一个完整的内核优化配置

net.ipv4.ip\_forward = 0

net.ipv4.conf.default.rp\_filter = 1

net.ipv4.conf.default.accept\_source\_route = 0

kernel.sysrq = 0

kernel.core\_uses\_pid = 1

net.ipv4.tcp\_syncookies = 1

kernel.msgmnb = 65536

kernel.msgmax = 65536

kernel.shmmax = 68719476736

kernel.shmall = 4294967296

net.ipv4.tcp\_max\_tw\_buckets = 6000

net.ipv4.tcp\_sack = 1

net.ipv4.tcp\_window\_scaling = 1

net.ipv4.tcp\_rmem = 4096        87380   4194304

net.ipv4.tcp\_wmem = 4096        16384   4194304

net.core.wmem\_default = 8388608

net.core.rmem\_default = 8388608

net.core.rmem\_max = 16777216

net.core.wmem\_max = 16777216

net.core.netdev\_max\_backlog = 262144



net.core.somaxconn = 262144

net.ipv4.tcp\_max\_orphans = 3276800

net.ipv4.tcp\_max\_syn\_backlog = 262144

net.ipv4.tcp\_timestamps = 0

net.ipv4.tcp\_synack\_retries = 1

net.ipv4.tcp\_syn\_retries = 1

net.ipv4.tcp\_tw\_recycle = 1

net.ipv4.tcp\_tw\_reuse = 1

net.ipv4.tcp\_mem = 94500000 915000000 927000000

net.ipv4.tcp\_fin\_timeout = 1

net.ipv4.tcp\_keepalive\_time = 30

net.ipv4.ip\_local\_port\_range = 1024   65000



 

