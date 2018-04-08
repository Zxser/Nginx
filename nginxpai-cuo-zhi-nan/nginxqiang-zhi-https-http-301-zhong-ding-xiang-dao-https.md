### 一、申请SSL证书

国外的startssl和国内的沃通都可以申请免费的SSL证书，当然也有很多收费的，如果只是个人博客网站，其实免费的足矣，可以参考文章：[沃通免费SSL证书申请](https://www.xiaoz.me/archives/6481)，申请免费的SSL证书。

### 二、相关环境准备

光有了证书还不行呀，您还需要搭建WEB服务器才能将证书放上去，上面已经提到LNMP一键包、AMH主机面板、OneinStack均使用的Nginx作为WEB服务器，安装其中之一即可。

### 三、部署SSL

最关键的步骤来了，首先您需要将步骤一中的SSL证书上传到服务器的某个目录，可以通过[winscp上传](http://www.xiaoz.me/archives/3250)，比如将.crt和.key文件上传到/usr/local/nginx/conf/ssl/目录，找到主机配置文件（通常是在/usr/local/nginx/conf/vhost/xxx.conf），使用vi编辑器再server段内加入下面几行：



```
listen 443 ssl;
ssl_certificate /usr/local/nginx/conf/ssl/www_xiaoz_me.crt;
ssl_certificate_key /usr/local/nginx/conf/ssl/www_xiaoz_me.key;
```

### 四、放行443端口

https（SSL）需要使用443端口，如果您的防火墙（iptables）没有放行443端口可能导致网站无法访问，请按照下面三个语句执行。



```
vi /etc/sysconfig/iptables   ##编辑配置文件
-A INPUT -m state --state NEW -m tcp -p tcp --dport 443 -j ACCEPT   ##（允许80端口通过防火墙）
/etc/init.d/iptables restart ##重启防火墙
```

### 五、重启nginx服务器

军哥的LNMP直接输入命令”lnmp nginx restart”，AMH输入命令”amh nginx restart”,另外如果您使用的startssl的证书在重启nginx的时候会要求输入证书密码，若配置错误可能导致nginx启动失败，修改主机配置文件前请先做好相关备份。

### 六、配置文件实例

下面这段代码是80端口（http）和443端口（https）共存实例，供参考。



```
server
{
listen 80;
listen 443 ssl;
server_name www.xiaoz.me;
index index.html index.htm index.php;
root /home/wwwroot/www.xiaoz.me/;
#ssl on; 这里要注释掉
ssl_certificate /usr/local/nginx/conf/ssl/www_iamle_com.crt;
ssl_certificate_key /usr/local/nginx/conf/ssl/www_iamle_com.key;
#以下配置省略
}
```



