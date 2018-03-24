NGINX虽然已经对静态内容做过优化。 但在高流量网站的情况下，仍然可以使用open\_file\_cache进一步提高性能。 NGINX缓存将最近使用的文件描述符和相关元数据（如修改时间，大小等）存储在缓存中。 缓存不会存储所请求文件的内容。

open\_file\_cache

启用此指令将存储以下信息的缓存：

打开的文件描述符和相关元数据，如大小，修改时间等

文件和目录的存在

与查找相关的任何错误，例如“权限被拒绝”，“文件未找到”等

缓存定义固定大小，并且在溢出期间，它移除最近最少使用（LRU）元素。 缓存在一段时间不活动之后逐出元素。 默认情况下禁用该指令。 如下例子：

    http{

       open\_file\_cache max=1000 inactive=20s;

       }

在上述配置中，为1,000个元素定义了一个缓存。 inactive参数配置到期时间为20秒。 没有必要为该指令设置非活动时间段，默认情况下，非活动时间段为60秒。

NGINX还定义了一些相关的指令，可用于在错误和有效性检查期间配置open\_file\_cache的行为。

open\_file\_cache\_valid

NGINX的open\_file\_cache保存信息的快照。 由于信息在源处更改，快照可能在一段时间后无效。 open\_file\_ cache\_valid指令定义时间段（以秒为单位），之后将重新验证open\_file\_cache中的元素。 如下例子：

    http{

       open\_file\_cache\_valid 30s;

       }

默认情况下，60秒后重新检查元素。

open\_file\_cache\_min\_uses

NGINX将在非活动时间段之后从高速缓存中清除元素。 此指令可用于配置最小访问次数以将元素标记为活动使用。 默认情况下，最小访问次数设置为1次或更多次。如下例子

    http{

       open\_file\_cache\_min\_uses 4;

       }

open\_file\_cache\_errors

如前所述，NGINX可以缓存在文件访问期间发生的错误。 但是这需要通过设置open\_file\_cache\_errors指令来启用。 如果启用错误缓存，则在访问资源（不查找资源）时，NGINX会报告相同的错误。

    http{

       open\_file\_cache\_errors on;

       }

默认情况下，错误缓存设置为关闭。







