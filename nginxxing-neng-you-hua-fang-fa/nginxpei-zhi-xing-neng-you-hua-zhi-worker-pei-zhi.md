配置Nginx workers



NGINX根据指定的配置运行固定数量的工作进程。 这些工作进程负责处理所有处理。 在下面的章节中，我们将调整NGINX worker参数。 这些参数是NGINX全局上下文的一部分。

worker\_processes

worker\_processes指令控制工作进程数：

    worker\_processes 1;

其默认值为1，这意味着NGINX只运行一个worker。 该值应根据可用内核数，磁盘，网络子系统，服务器负载等更改为最佳值。

我们可以将值设置为可用的核心数。 使用lscpu确定可用的核心数：

    $ lscpu

    Architecture:  x86\_64

    CPU op-mode\(s\):  32-bit, 64-bit

    Byte Order:  Little Endian

    CPU\(s\):        4

同样可以通过grep cpuinfo得到：

    $ cat /proc/cpuinfo \| grep 'processor' \| wc -l

现在我们设置worker数为4:

    \# One worker per CPU-core.

       worker\_processes 4;

或者，可以将其设置为auto。 这样nginx会自动根据核心数为生成对应数量的worker进程。

accept\_mutex

由于我们在NGINX中配置了多个workers，因此我们还应配置影响worker的相关指令。 events区域下accept\_mutex参数将使每个可用的worker进程逐个接受新连接。 默认情况下，该标志设置为on。 如：

    events {

         accept\_mutex on;

    }

如果accept\_mutex为off，所有可用的worker将从等待状态唤醒，但只有一个worker处理连接。 这导致惊群现象，每秒重复多次。 这种现象导致服务器性能下降，因为所有被唤醒的worker都在占用CPU时间。 这导致增加了非生产性CPU周期和未使用的上下文切换。

accept\_mutex\_delay

当启用accept\_mutex时，只有一个具有互斥锁的worker程序接受连接，而其他工作程序则轮流等待。 accept\_mutex\_delay对应于worker等待的时间帧，然后它尝试获取互斥锁并开始接受新的连接。 默认值为500毫秒

    events{

         accept\_mutex\_delay 500ms;

     }

worker\_connections

下一个要查看的配置是worker\_connections，默认值为512.该指令设置worker进程最大打开的连接数:

    events{

          worker\_connections 512;

    }

将worker\_connections增加到1024或更高的值，以允许同时处理更多连接。

worker\_rlimit\_nofile

同时连接的数量受限于系统上可用的文件描述符的数量，因为每个套接字将打开一个文件描述符。 如果NGINX尝试打开比可用文件描述符更多的套接字，会发现error.log中出现Too many opened files的信息。

使用ulimit检查文件描述符的数量：

    $ ulimit -n

现在，将此值增加到大于worker\_processes \* worker\_connections的值。 应该是增加当前worker运行用户的最大文件打开数值。

NGINX提供了worker\_rlimit\_nofile指令，这是除了ulimit的一种设置可用的描述符的方式。 该指令与使用ulimit对用户的设置是同样的效果。此指令的值将覆盖ulimit的值，如：

    worker\_rlimit\_nofile 20960;

multi\_accept

multi\_accept指令使得NGINX worker能够在获得新连接的通知时尽可能多的接受连接。 此指令的作用是立即接受所有连接放到监听队列中。 如果指令被禁用，worker进程将逐个接受连接。

    events{

          multi\_accept on;

    }









