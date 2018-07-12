

**由于默认的Linux内核参数考虑的是最通用的场景，这明显不符合用于支持高并发访问的Web服务器的定义，所以需要修改Linux内核参数，使得Nginx可以拥有更高的性能。**

**在优化内核时，可以做的事件很多，不过，我们通常会根据业务特点来进行调整，当Nginx作为静态Web内容服务器、反向代理服务器或是提供图片缩略功能（实时压缩图片）的服务器时，其内核参数的调整都是不同的。这里只针对最通用的、使Nginx支持更多并发请求的TCP网络参数做简单说明。**

**首先，需要修改/etc/sysctl.conf来更改内核参数，例如，最常用的配置：**

1. \#原有字段  

2. net.ipv4.tcp\_syncookies = 1  

3. \#新增字段  

4. fs.file-max = 999999  

5. net.ipv4.tcp\_tw\_reuse = 1  

6. net.ipv4.tcp\_keepalive\_time = 600  

7. net.ipv4.tcp\_fin\_timeout = 30  

8. net.ipv4.tcp\_max\_tw\_buckets = 5000  

9. net.ipv4.ip\_local\_port\_range = 1024 61000  

10. net.ipv4.tcp\_rmem = 10240 87380 12582912  

11. net.ipv4.tcp\_wmem = 10240 87380 12582912  

12. net.core.netdev\_max\_backlog = 8096  

13. net.core.rmem\_default = 6291456  

14. net.core.wmem\_default = 6291456  

15. net.core.rmem\_max = 12582912  

16. net.core.wmem\_max = 12582912  

17. net.ipv4.tcp\_max\_syn\_backlog = 1024  

  


**然后执行sysctl -p命令，使上述参数生效。**

**上面的参数意义解释如下：**

**fs.file-max = 999999：这个参数表示进程（比如一个worker进程）可以同时打开的最大句柄数，这个参数直线限制最大并发连接数，需根据实际情况配置。  
**

**net.ipv4.tcp\_tw\_reuse = 1：这个参数设置为1，表示允许将TIME-WAIT状态的socket重新用于新的TCP连接，这对于服务器来说很有意义，因为服务器上总会有大量TIME-WAIT状态的连接。  
**

**net.ipv4.tcp\_keepalive\_time = 600：这个参数表示当keepalive启用时，TCP发送keepalive消息的频度。默认是2小时，若将其设置的小一些，可以更快地清理无效的连接。  
**

**net.ipv4.tcp\_fin\_timeout = 30：这个参数表示当服务器主动关闭连接时，socket保持在FIN-WAIT-2状态的最大时间。  
**

**net.ipv4.tcp\_max\_tw\_buckets = 5000：这个参数表示操作系统允许TIME\_WAIT套接字数量的最大值，如果超过这个数字，TIME\_WAIT套接字将立刻被清除并打印警告信息。该参数默认为180 000，过多的TIME\_WAIT套接字会使Web服务器变慢。  
**

**net.ipv4.tcp\_max\_syn\_backlog = 1024：这个参数标示TCP三次握手建立阶段接受SYN请求队列的最大长度，默认为1024，将其设置得大一些可以使出现Nginx繁忙来不及accept新连接的情况时，Linux不至于丢失客户端发起的连接请求。  
**

**net.ipv4.ip\_local\_port\_range = 1024 61000：这个参数定义了在UDP和TCP连接中本地（不包括连接的远端）端口的取值范围。  
**

**net.ipv4.tcp\_rmem = 10240 87380 12582912：这个参数定义了TCP接受缓存（用于TCP接受滑动窗口）的最小值、默认值、最大值。  
**

**net.ipv4.tcp\_wmem = 10240 87380 12582912：这个参数定义了TCP发送缓存（用于TCP发送滑动窗口）的最小值、默认值、最大值。  
**

**net.core.netdev\_max\_backlog = 8096：当网卡接受数据包的速度大于内核处理的速度时，会有一个队列保存这些数据包。这个参数表示该队列的最大值。  
**

**net.core.rmem\_default = 6291456：这个参数表示内核套接字接受缓存区默认的大小。  
**

**net.core.wmem\_default = 6291456：这个参数表示内核套接字发送缓存区默认的大小。  
**

**net.core.rmem\_max = 12582912：这个参数表示内核套接字接受缓存区的最大大小。  
**

**net.core.wmem\_max = 12582912：这个参数表示内核套接字发送缓存区的最大大小。  
**

**net.ipv4.tcp\_syncookies = 1：该参数与性能无关，用于解决TCP的SYN\*\*\*。  
**

**注意：滑动窗口的大小与套接字缓存区会在一定程度上影响并发连接的数目。每个TCP连接都会为维护TCP滑动窗口而消耗内存，这个窗口会根据服务器的处理速度收缩或扩张。**

**参数net.core.wmem\_max = 12582912的设置，需要平衡物理内存的总大小、Nginx并发处理的最大连接数量而确定。当然，如果仅仅为了提供并发量使服务器不出现Out Of Memory问题而去降低滑动窗口大小，那么并不合适，因为滑动窗过小会影响大数据量的传输速度。net.core.rmem\_default = 6291456、net.core.wmem\_default = 6291456、**

**net.core.rmem\_max = 12582912和net.core.wmem\_max = 12582912这4个参数的设置需要根据我们的业务特性以及实际的硬件成本来综合考虑。  
**

**Nginx并发处理的最大连接量：由nginx.conf中的work\_processes和work\_connections参数决定。**

