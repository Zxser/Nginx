在nginx里面加一个location就可以了，具体设置如下：

location ~ ^/status$ {

                include fastcgi\_params;

                fastcgi\_pass 127.0.0.1:9000;

                fastcgi\_param SCRIPT\_FILENAME $fastcgi\_script\_name;

        }

然后在php-fpm.conf里面打开选项

pm.status\_path = /status

这样的话通过http://域名/status就可以看到当前的php情况，以前之知道可以配置location来看nginx的状态，没想到还可以看php-fpm的状态，，真的是学习了，，看到的状态如下：

pool: www php运行的组

process manager: dynamic php-fpm运行的方式

start time: 04/Jun/2012:16:05:32 +0800 开始时间

start since: 5932

accepted conn: 65678 接受链接

listen queue: 0 监听队列

max listen queue: 1 最大监听队列

listen queue len: 128 监听队列len

idle processes: 82 空闲进程

active processes: 4 活动进程

total processes: 86 总进程

max active processes: 25 最大活动进程

max children reached: 0 最大的子进程达到

有了这个，就可以实时的查看php-fpm的状态，进而优化php-fpm



