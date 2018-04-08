本文讲介绍

使用sls安装nginx，并管理nginx的配置文件，当nginx配置文件被修改时，自动更新配置文件，并重启nginx

在master端上配置nginx.sls文件

&lt;!--  
 /\* Font Definitions \*/  
 @font-face  
	{font-family:宋体;  
	panose-1:2 1 6 0 3 1 1 1 1 1;  
	mso-font-alt:SimSun;  
	mso-font-charset:134;  
	mso-generic-font-family:auto;  
	mso-font-pitch:variable;  
	mso-font-signature:3 680460288 22 0 262145 0;}  
@font-face  
	{font-family:"Cambria Math";  
	panose-1:2 4 5 3 5 4 6 3 2 4;  
	mso-font-charset:1;  
	mso-generic-font-family:roman;  
	mso-font-format:other;  
	mso-font-pitch:variable;  
	mso-font-signature:0 0 0 0 0 0;}  
@font-face  
	{font-family:Calibri;  
	panose-1:2 15 5 2 2 2 4 3 2 4;  
	mso-font-charset:0;  
	mso-generic-font-family:swiss;  
	mso-font-pitch:variable;  
	mso-font-signature:-536870145 1073786111 1 0 415 0;}  
@font-face  
	{font-family:"\@宋体";  
	panose-1:2 1 6 0 3 1 1 1 1 1;  
	mso-font-charset:134;  
	mso-generic-font-family:auto;  
	mso-font-pitch:variable;  
	mso-font-signature:3 680460288 22 0 262145 0;}  
 /\* Style Definitions \*/  
 p.MsoNormal, li.MsoNormal, div.MsoNormal  
	{mso-style-unhide:no;  
	mso-style-qformat:yes;  
	mso-style-parent:"";  
	margin:0cm;  
	margin-bottom:.0001pt;  
	text-align:justify;  
	text-justify:inter-ideograph;  
	mso-pagination:none;  
	font-size:10.5pt;  
	mso-bidi-font-size:11.0pt;  
	font-family:"Calibri","sans-serif";  
	mso-ascii-font-family:Calibri;  
	mso-ascii-theme-font:minor-latin;  
	mso-fareast-font-family:宋体;  
	mso-fareast-theme-font:minor-fareast;  
	mso-hansi-font-family:Calibri;  
	mso-hansi-theme-font:minor-latin;  
	mso-bidi-font-family:"Times New Roman";  
	mso-bidi-theme-font:minor-bidi;  
	mso-font-kerning:1.0pt;}  
.MsoChpDefault  
	{mso-style-type:export-only;  
	mso-default-props:yes;  
	font-family:"Calibri","sans-serif";  
	mso-bidi-font-family:"Times New Roman";  
	mso-bidi-theme-font:minor-bidi;}  
 /\* Page Definitions \*/  
 @page  
	{mso-page-border-surround-header:no;  
	mso-page-border-surround-footer:no;}  
@page WordSection1  
	{size:595.3pt 841.9pt;  
	margin:72.0pt 90.0pt 72.0pt 90.0pt;  
	mso-header-margin:42.55pt;  
	mso-footer-margin:49.6pt;  
	mso-paper-source:0;  
	layout-grid:15.6pt;}  
div.WordSection1  
	{page:WordSection1;}  
--&gt;  


mkdir -p /srv/salt/nginx

cd /srv/salt/nginx/

vim init.sls

nginx:

 pkg:

 - installed

 service:

 - running

 - enable: True

 - reload: True

 - watch:

 - pkg: nginx

 - file: /etc/nginx/nginx.conf

 - file: /etc/nginx/conf.d/default.conf

/etc/nginx/nginx.conf:

 file.managed:

 - source: salt://etc/nginx/nginx.conf

 - user: root

 - group: root

 - mode: 644

/etc/nginx/conf.d/default.conf:

 file.managed:

 - source: salt://etc/nginx/conf.d/default.conf

 - user: root

 - group: root

 - mode: 644

文件讲解

• nginx: 这是要安装的包名，也是sls文件的id,不能重复

• pkg: pkg是包管理模块，对应/usr/lib/python2.6/site-packages/salt/states下的模块pkg.py

• installed installed是pkg模块下的函数，id\(nginx\)作为installed的参数进行调用

• service: service是服务模块，对应/usr/lib/python2.6/site-packages/salt/states下的模块service.py, 由于service是一个key,其下的running, require, watch是列表形式的值，因此service之后有冒号

• running running是service.py模块下的函数，id\(nginx\)作为running的参数进行调用

• enable: True

• reload: True

• watch: watch: 表示对文件$file的监控，当master 向minion传递$file时，新的$file与minion上原有文件不一致时，会重启nginx服务

• pkg: nginx

• file: /etc/nginx/nginx.conf

• file: /etc/nginx/conf.d/default.conf

• /etc/nginx/nginx.conf: 这一行同样是id不能重复：表示传递到minion时所处的位置，同时也作为file.managed函数的参数

• file.managed:

• file.py模块的managed函数，下面的source，user, group, mode都是managed函数的参数

• source: salt://etc/nginx/nginx.conf

• source 是managed函数的参数，指定要传递到minion端的源文件. –

• salt://etc/nginx/nginx.conf 表示/etc/nginx/nginx.conf在/srv/salt之下，/srv/salt是saltstack的根目录

• user: root

表示文件的属主

• group: root

表示文件的属组

• mode: 644

表示文件的权限

开始配置

1:在master端上安装nginx，方便生成nginx的配置文件

 yum -y install nginx

2:创建nginx同步目录

mkdir /srv/salt/etc/nginx/conf.d -p

3:拷贝nginx的配置文件到/srv/salt/etc/nginx/目录下

cp /etc/nginx/nginx.conf /srv/salt/etc/nginx/

4:拷贝default.conf配置文件到/srv/salt/nginx/conf.d/目录下

cp /etc/nginx/conf.d/default.conf /srv/salt/etc/nginx/conf.d/

5:开始安装

 salt '\*' state.sls nginx

6:测试是否安装成功

salt '\*' cmd.run 'rpm -qa \| grep nginx'

接下来实现配置更新

手动更新配置文件

在master端将默认端口更改为8080

vim /srv/salt/etc/nginx/conf.d/default.conf

listen       8080 default\_server;

在minion端执行指令，观察

salt-call state.sls nginx

自动更新配置文件

定义pillar的主目录，同时创建pillar目录（master端）

vim /etc/salt/master   \#找到以下内容取消注释

pillar\_roots:

  base:

    - /srv/pillar

pillar\_opts: True

mkdir -p /srv/pillar

定义入口文件top.sls

入口文件的作用一般是定义pillar的数据覆盖被控主机的有效范围，’\*’代表任意主机,默认从 base 标签开始解析执行,下一级是操作的目标

cat /srv/pillar/top.sls

base:

  '\*':

    - nginx    \#指代的是nginx.sls文件

定义nginx文件,每分钟更新一次

install -d /srv/pillar/nginx

cd nginx/

cat init.sls

schedule:

 nginx:

    function: state.sls

    minutes: 1

    args:

        - 'nginx'

刷新被控主机的pillar信息

salt '\*' saltutil.refresh\_pillar

查看上面定义的nginx.sls数据项,出现以下内容表示成功

salt '\*' pillar.data

192.168.31.166:

    ----------

    schedule:

        ----------

        nginx:

            ----------

            args:

                - nginx

            function:

                state.sls

            minutes:

                1

192.168.31.188:

    ----------

    schedule:

        ----------

        nginx:

            ----------

            args:

                - nginx

            function:

                state.sls

            minutes:

                1

测试

在master端将默认端口更改为666

vim /srv/salt/etc/nginx/conf.d/default.conf

 listen       666 default\_server;

一分钟后在minion端查看端口：

netstat -tnl



