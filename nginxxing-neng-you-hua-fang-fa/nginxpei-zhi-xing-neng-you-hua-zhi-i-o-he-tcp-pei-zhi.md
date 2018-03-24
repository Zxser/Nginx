配置Nginx I/O

Sendfile

当应用程序传输文件时，内核首先缓冲数据，然后将数据发送到应用程序缓冲区。 应用程序反过来将数据发送到目的地。 Sendfile方法是一种改进的数据传输方法，其中数据在操作系统内核空间内的文件描述符之间复制，而不将数据传输到应用程序缓冲区。 这使操作系统资源的利用率提高。

可以使用sendfile指令启用该方法。 该指令可用于http，server和location代码块：

```
http{

     sendfile on;

}
```

此指令默认为off。

直接I/O

操作系统内核通常尝试优化和缓存任何读/写请求。 由于数据在内核中缓存，对同一位置的任何后续读取请求将更快，因为不需要再从磁盘读取信息。

直接I/O是文件系统的一个功能，其从应用程序到磁盘直接读取和写入，从而绕过所有操作系统缓存。 这使得更好地利用CPU周期和提高缓存效率。

该方法用于数据具有较差命中率的地方。 这样的数据不需要在任何高速缓存中，并且可以在需要时加载。 它可以用于提供大文件。 directio指令启用该功能。 该指令可用于http，server和location区块：

```
location /video/ {

     directio 4m;

}
```

任何大于指令中指定的文件将由直接I/O加载。 其它情况下禁用此参数。

直接I/O取决于执行数据传输时的块大小。 NGINX有directio\_alignment指令来设置块大小。 该指令可用于http，server和location区块：

```
location /video/ {

     directio 4m;

     directio\_alignment 512;

}
```

除了XFS文件系统，默认值512字节在所有Linux版本运行良好。在此文件系统下，大小应增加到4 KB。

异步I/O

异步I/O允许进程进行不受阻塞或不需要等待I/O完成的I/O操作。

aio命令可在NGINX配置的http，server和location区块下使用。 根据在指令所在区块，该指令将为匹配的请求执行异步I/O。 该参数适用于Linux内核2.6.22+和FreeBSD 4.3。 如下代码：

```
location /data {

     aio on;

}
```

默认情况下，该参数设置为off。 在Linux上，aio需要启用direction，而在FreeBSD上，sendfile需要禁用以使aio生效。

该指令具有特殊的线程值，可以为发送和读操作启用多线程。 多线程支持仅在Linux平台上可用，并且只能与处理请求的epoll，kqueue或eventport方法一起使用。

为了使用线程值，在编译Nginx时使用–with-threads选项配置多线程支持。 在NGINX全局上下文中使用thread\_pool指令添加一个线程池。 在aio配置中使用该线程池：

```
thread\_pool io\_pool threads=16;

   http{

   ........

      location /data{

        sendfile    on;

        aio        threads=io\_pool;

} }
```

配置Nginx TCP

HTTP是一种基于应用的协议，它使用TCP作为传输层。 在TCP中，数据以称为TCP分组的块的形式传送。 NGINX提供了改变底层TCP栈的行为的指令。 这些参数更改了单个套接字连接的属性。

TCP\_NODELAY

TCP/IP网络存在“小包”问题，其中单字符消息可能在高负载网络上导致网络拥塞。 例如分组大小为41字节，其中40字节用于TCP报头，只有1字节是有用信息。 这些小包占用了大约4000％的巨大开销并且使得网络饱和。

ohn Nagle通过不立即发送小包来解决问题（Nagle的算法）。 所有这样的分组被收集一定量的时间，然后作为单个分组一次发送。 这改进了底层网络的的效率。 因此，典型的TCP/IP协议栈在将数据包发送到客户端之前需要等待200毫秒。

在打开套接字时可以使用TCP\_NODELAY选项来禁用Nagle的缓冲算法，并在数据可用时立即发送。 NGINX提供了tcp\_nodelay指令来启用此选项。 该指令可用于http，server和location区块：

```
http{

     tcp\_nodelay on;

}
```

该指令默认情况下启用。

TCP\_CORK

作为Nagle算法的替代方案，Linux提供了TCP\_CORK选项。 该选项告诉TCP堆栈附加数据包，并在它们已满或当应用程序通过显式删除TCP\_CORK指示发送数据包时发送它们。 这使得发送的数据分组是最优量，并且因此提高了网络的效率。

NGINX提供了tcp\_nopush指令，在连接套接字时启用TCP\_CORK。 该指令可用于http，server和location区块：

```
http{

     tcp\_nopush on;

}
```



