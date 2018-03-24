请求缓冲区在NGINX请求处理中起着重要作用。 在接收到请求时，NGINX将其写入这些缓冲区。 这些缓冲区中的数据可作为NGINX变量使用，例如$request\_body。 如果缓冲区与请求大小相比较小，则数据将写入磁盘上的文件，因此将涉及I/O操作。 NGINX提供了可以改变请求缓冲区的各种指令。

client\_body\_buffer\_size



此指令设置用于请求主体的缓冲区大小。 如果主体超过缓冲区大小，则完整主体或其一部分将写入临时文件。 如果NGINX配置为使用文件而不是内存缓冲区，则该指令会被忽略。 默认情况下，该指令为32位系统设置一个8k缓冲区，为64位系统设置一个16k缓冲区。 该指令在NGINX配置的http，server和location区块使用。如下：

    server{

          client\_body\_buffer\_size 8k;

    }

client\_max\_body\_size



此指令设置NGINX能处理的最大请求主体大小。 如果请求大于指定的大小，则NGINX发回HTTP 413（Request Entity too large）错误。 如果服务器处理大文件上传，则该指令非常重要。

默认情况下，该指令值为1m。 如下：

    server{

          client\_max\_body\_size 2m;

    }

client\_body\_in\_file\_only



此指令禁用NGINX缓冲区并将请求体存储在临时文件中。 文件包含纯文本数据。 该指令在NGINX配置的http，server和location区块使用。 可选值有：

off:该值将禁用文件写入

clean：请求body将被写入文件。 该文件将在处理请求后删除。

on: 请求正文将被写入文件。 处理请求后，将不会删除该文件。

默认情况下，指令值为关闭。 如下：

    http{

          client\_body\_in\_file\_only clean;

    }

client\_body\_in\_single\_buffer



该指令设置NGINX将完整的请求主体存储在单个缓冲区中。 默认情况下，指令值为off。 如果启用，它将优化读取$request\_body变量时涉及的I/O操作。如下例子：

    server{

          client\_body\_in\_single\_buffer on;

    }

client\_body\_temp\_path



此指令指定存储请求正文的临时文件的位置。 除了位置之外，指令还可以指定文件是否需要最多三个级别的文件夹层次结构。 级别指定为用于生成文件夹的位数。

默认情况下，NGINX在NGINX安装路径下的client\_body\_temp文件夹创建临时文件。 如下例子：

    server{

          client\_body\_temp\_pathtemp\_files 1 2;

          }

该指令生成的文件路径如temp\_files/1/05/0000003051。

client\_header\_buffer\_size



此指令与client\_body\_buffer\_size类似。 它为请求头分配一个缓冲区。 如果请求头大小大于指定的缓冲区，则使用large\_client\_header\_buffers指令分配更大的缓冲区。如下例子：

    http{

          client\_header\_buffer\_size 1m;

          }

large\_client\_header\_buffers



此指令规定了用于读取大型客户端请求头的缓冲区的最大数量和大小。 这些缓冲区仅在缺省缓冲区不足时按需分配。 当处理请求或连接转换到保持活动状态时，释放缓冲区。如下例子：

    http{

          large\_client\_header\_buffers 4 8k;

          }





