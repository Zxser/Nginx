### 安装准备

Fikker需要监听`80/443/6780`端口，若您的服务器已经安装过Nginx/Apache等服务，需要先停用，推荐使用一台全新的VPS进行安装，如果您的域名没有备案可以考虑《[野草云香港VPS](https://www.xiaoz.me/archives/8394)》

### 开始安装

依次复制下面的命令（需要root用户）执行：

```
#安装wget，若已经安装这一步可省略

yum -y install wget 

#下载安装包

wget -c http://www.fikker.com/dl/fikkerd-3.7.3-linux-x86-64.tar.gz

#解压

tar zxvf fikkerd-3.7.3-linux-x86-64.tar.gz

#进入安装目录
cd
 fikkerd-3.7.3-linux-x86-64

#运行Fikker

./fikkerd.sh start

```

上面已经提到，Fikker监听`80/443/6780`端口，请注意在防火墙放行端口,输入下面的命令：

```
#如果防火墙使用的iptables（Centos 6）

iptables -I INPUT -p tcp --dport 80 -j ACCEPT
iptables -I INPUT -p tcp --dport 443 -j ACCEPT
iptables -I INPUT -p tcp --dport 6780 -j ACCEPT
service iptables save
service iptables restart

#如果使用的是firewall（CentOS 7）

firewall-cmd --zone=public --add-port=80/tcp --permanent 
firewall-cmd --zone=public --add-port=443/tcp --permanent 
firewall-cmd --zone=public --add-port=6780/tcp --permanent 
firewall-cmd --reload

```

完成后访问`http://IP:6780`，初始密码为`123456`，如果打不开，请输入命令`netstat -apn|grep '6780'`查看端口是否监听，检查防火墙是否放行端口。

![](https://cdn.xiaoz.me/wp-content/uploads/2017/06/fikker-login.png)

### 添加站点

在Fikker 后台 – 管理工具 – 主机管理 – 右下角添加主机，添加一个需要CDN加速的域名（支持HTTP/HTTPS），如下截图。

![](https://cdn.xiaoz.me/wp-content/uploads/2017/06/fikker_add.png)

![](https://cdn.xiaoz.me/wp-content/uploads/2017/06/add_xiaoz.png)

### 设置源站

添加主机后，还需要设置回源地址，告知CDN节点从哪里获取数据，源站添加完毕后大功告成，您可以将DNS解析至CDN节点了，推荐使用《[免费智能DNS解析CloudXNS](https://www.xiaoz.me/archives/6569)》，这样可实现分区域解析。

![](https://cdn.xiaoz.me/wp-content/uploads/2017/06/add_yuan.png)

### 其它操作

如果您需要将Fikker注册为服务，请执行下面的命令：

```
#注册服务

./fikkerd.sh install

#停止服务

./fikkerd.sh stop

#删除服务

./fikkerd.sh uninstall
```



