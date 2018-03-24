php-fpm进程池开启进程有两种方式，一种是static，直接开启指定数量的php-fpm进程，不再增加或者减少；

另一种则是dynamic，开始时开启一定数量的php-fpm进程，当请求量变大时，动态的增加php-fpm进程数到上限，

当空闲时自动释放空闲的进程数到一个下限。这两种不同的执行方式，可以根据服务器的实际需求来进行调整。

要用到的一些参数，分别是pm、pm.max\_children、pm.start\_servers、pm.min\_spare\_servers和pm.max\_spare\_servers。

pm表示使用那种方式，有两个值可以选择，就是static（静态）或者dynamic（动态）。

下面4个参数的意思分别为：

• pm.max\_children：静态方式下开启的php-fpm进程数量，在动态方式下他限定php-fpm的最大进程数（这里要注意pm.max\_spare\_servers的值只能小于等于pm.max\_children）

• pm.start\_servers：动态方式下的起始php-fpm进程数量。

• pm.min\_spare\_servers：动态方式空闲状态下的最小php-fpm进程数量。

• pm.max\_spare\_servers：动态方式空闲状态下的最大php-fpm进程数量。如果dm设置为static，那么其实只有pm.max\_children这个参数生效。系统会开启参数设置数量的php-fpm进程。php-fpm一个进程大概会占20m-40m的内存，所以他的数字大小的设置要根据你的物理内存的大小来设置，还要注意到其他的内存占用，如数据库，系统进程等，来确定以上4个参数的设定值！

如果dm设置为dynamic，4个参数都生效。系统会在php-fpm运行开始时启动pm.start\_servers个php-fpm进程，

然后根据系统的需求动态在pm.min\_spare\_servers和pm.max\_spare\_servers之间调整php-fpm进程数。

参数要求pm.start\_servers的值在pm.min\_spare\_servers和pm.max\_spare\_servers之间。

例：主机内存为1.6G，默认lnmp设置为20，10，10，20；改为如下25，5，5，25

因为网站访客人数很少，所以初始php-fpm线程5个就好了，不会占用过多内存，当访客突然增加，也会动态调整直到最高的限值25.

\[www\]

listen = /tmp/php-cgi.sock

listen.backlog = -1

listen.allowed\_clients = 127.0.0.1

listen.owner = www

listen.group = www

listen.mode = 0666

user = www

group = www

pm = dynamic

pm.max\_children = 25

pm.start\_servers = 5

pm.min\_spare\_servers = 5

pm.max\_spare\_servers = 25

request\_terminate\_timeout = 100

request\_slowlog\_timeout = 0

slowlog = var/log/slow.log



