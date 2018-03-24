启用FastCGI缓存



编辑必须启用缓存的虚拟主机配置文件。

    nano /etc/nginx/sites-enabled/vhost

将以下行添加到server{}指令之外的文件顶部：

    fastcgi\_cache\_path /etc/nginx/cache levels=1:2 keys\_zone=MYAPP:100m inactive=60m;

    fastcgi\_cache\_key "$scheme$request\_method$host$request\_uri";

“fastcgi\_cache\_path”指令指定缓存（/etc/nginx/cache）的位置，其大小（100m），内存区域名称（MYAPP），子目录级别和非活动定时器。

位置可以在硬盘上的任何地方; 但是，大小必须小于您的服务器的RAM +交换，否则你会收到一个错误，“无法分配内存”。 如果缓存在“inactive”选项指定的特定时间内没有被访问（这里为60分钟），Nginx将删除它。

“fastcgi\_cache\_key”指令指定如何哈希缓存文件名。 Nginx基于此指令使用MD5加密访问的文件。

在location ~ .php$ { }里添加如下行：

    fastcgi\_cache MYAPP;

    fastcgi\_cache\_valid 200 60m;

“fastcgi\_cache”指令引用我们在“fastcgicache\_path”指令中指定的内存区域名称，并将缓存存储在此区域中。

默认情况下，Nginx根据这些响应头里指定的时间决定存储缓存对象的时间：X-Accel-Expires / Expires / Cache-Control。

如果缺少这些头，“fastcgi\_cache\_valid”指令用于指定默认缓存生命周期。 在上面输入的语句中，只缓存状态代码为200的响应。 也可以指定其他响应代码。

测试配置：

    service nginx configtest

重载Nginx:

    service nginx reload

完整的vhost配置文件如下：

    fastcgi\_cache\_path /etc/nginx/cache levels=1:2 keys\_zone=MYAPP:100m inactive=60m;

    fastcgi\_cache\_key "$scheme$request\_method$host$request\_uri";

     

    server {

        listen   80;

     

        root /usr/share/nginx/html;

        index index.php index.html index.htm;

     

        server\_name example.com;

     

        location / {

            try\_files $uri $uri/ /index.html;

        }

     

        location ~ \.php$ {

            try\_files $uri =404;

            fastcgi\_pass unix:/var/run/php5-fpm.sock;

            fastcgi\_index index.php;

            include fastcgi\_params;

            fastcgi\_cache MYAPP;

            fastcgi\_cache\_valid 200 60m;

        }

    }

测试FastCGI缓存是否生效



创建/usr/share/nginx/html/time.php，内容如下：

    &lt;?php

    echo time\(\);

    ?&gt;

使用curl或您的Web浏览器多次请求此文件。

    root@droplet:~\# curl http://localhost/time.php;echo

    1382986152

    root@droplet:~\# curl http://localhost/time.php;echo

    1382986152

    root@droplet:~\# curl http://localhost/time.php;echo

    1382986152

如果缓存工作正常，您应该在缓存响应时在所有请求上看到相同的时间戳。

执行缓存位置的递归列表以查找此请求的缓存。

root@droplet:~\# ls -lR /etc/nginx/cache/

/etc/nginx/cache/:

total 0

drwx—— 3 www-data www-data 60 Oct 28 18:53 e

/etc/nginx/cache/e:

total 0

drwx—— 2 www-data www-data 60 Oct 28 18:53 18

/etc/nginx/cache/e/18:

total 4

-rw——- 1 www-data www-data 117 Oct 28 18:53 b777c8adab3ec92cd43756226caf618e

我们还可以使Nginx为响应添加一个“X-Cache”头，指示缓存是否被丢失或命中。

在server{}指令上面添加以下内容：

    add\_header X-Cache $upstream\_cache\_status;

重新加载Nginx服务，并使用curl执行详细请求以查看新标题。

root@droplet:~\# curl -v http://localhost/time.php

\* About to connect\(\) to localhost port 80 \(\#0\)

\* Trying 127.0.0.1…

\* connected

\* Connected to localhost \(127.0.0.1\) port 80 \(\#0\)

&gt; GET /time.php HTTP/1.1

&gt; User-Agent: curl/7.26.0

&gt; Host: localhost

&gt; Accept: \*/\*

&gt;

\* HTTP 1.1 or later with persistent connection, pipelining supported

&lt; HTTP/1.1 200 OK &lt; Server: nginx &lt; Date: Tue, 29 Oct 2013 11:24:04 GMT &lt; Content-Type: text/html &lt; Transfer-Encoding: chunked &lt; Connection: keep-alive &lt; X-Cache: HIT &lt; \* Connection \#0 to host localhost left intact 1383045828\* Closing connection \#0

不需要缓存的页面



某些动态内容（例如认证所需页面）不应缓存。 可以基于诸如“requesturi”，“requestmethod”和“http\_cookie”的服务器变量来排除这样的内容被高速缓存。

如下例子:

    \#Cache everything by default

    set $no\_cache 0;

     

    \#Don't cache POST requests

    if \($request\_method = POST\)

    {

        set $no\_cache 1;

    }

     

    \#Don't cache if the URL contains a query string

    if \($query\_string != ""\)

    {

        set $no\_cache 1;

    }

     

    \#Don't cache the following URLs

    if \($request\_uri ~\* "/\(administrator/\|login.php\)"\)

    {

        set $no\_cache 1;

    }

     

    \#Don't cache if there is a cookie called PHPSESSID

    if \($http\_cookie = "PHPSESSID"\)

    {

        set $no\_cache 1;

    }

要将“$no\_cache”变量应用到相应的指令，请将以下行放在location〜.php $ {}中，

    fastcgi\_cache\_bypass $no\_cache;

    fastcgi\_no\_cache $no\_cache;

“fasctcgicachebypass”指令忽略之前由我们设置的条件相关的请求的现有缓存。 如果满足指定的条件，“fastcginocache”指令不缓存请求。

清除缓存



缓存的命名约定基于我们为“fastcgicachekey”指令设置的变量。

    fastcgi\_cache\_key "$scheme$request\_method$host$request\_uri";

根据这些变量，当我们请求“http//localhost/time.php”时，以下是实际值：

    fastcgi\_cache\_key "httpGETlocalhost/time.php";

将此字符串传递到MD5哈希将输出以下字符串：

b777c8adab3ec92cd43756226caf618e

这就是高速缓存的文件名，就像我们输入的“levels = 1：2”子目录。 因此，目录的第一级将从这个MD5字符串的最后一个字符命名为1个字符，即e; 第二级将具有在第一级之后的最后2个字符，即18.因此，该高速缓存的整个目录结构如下：

/etc/nginx/cache/e/18/b777c8adab3ec92cd43756226caf618e

基于这种缓存命名格式，您可以用您最喜欢的语言开发一个清除脚本。 对于本教程，我将提供一个简单的PHP脚本，它清除POSTed URL的缓存。

/usr/share/nginx/html/purge.php：

    &lt;?php

    $cache\_path = '/etc/nginx/cache/';

    $url = parse\_url\($\_POST\['url'\]\);

    if\(!$url\)

    {

        echo 'Invalid URL entered';

        die\(\);

    }

    $scheme = $url\['scheme'\];

    $host = $url\['host'\];

    $requesturi = $url\['path'\];

    $hash = md5\($scheme.'GET'.$host.$requesturi\);

    var\_dump\(unlink\($cache\_path . substr\($hash, -1\) . '/' . substr\($hash,-3,2\) . '/' . $hash\)\);

    ?&gt;

向此文件发送带需要清除的URL的POST请求。

    curl -d 'url=http://www.example.com/time.php' http://localhost/purge.php

该脚本将根据是否清除缓存而输出true或false。 请确保从高速缓存中排除此脚本，并限制访问。





