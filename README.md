![三次技术升级][image-1]

男性的成长，总会伴随着信息搜集能力的升级。以我为例，从儿时的懵懂无知，到少年时的初尝禁果，再到现在的深藏功与名，一路走来，在下载某类型视频上经历了三次重大的技术升级。这三次技术升级堪比人类历史上的三次工业革命，在漫漫黑夜中点亮了我的生命，让我渐渐发现绚丽多彩的人生。

高三那年，在同学的MP4中我发现了新世界的大门，但直到高考结束后，我才正式吹响了向老司机进军的号角。在最初的懵懂岁月中，我一直靠某播苟延残喘，通过附近的视频功能如饥似渴地观摩新的作品。好景不长，随着王铁匠的入狱，某播伴随着的苦涩青春也一去不返。当上帝关了一扇门，一定会为你打开一扇窗，在我还有如无头苍蝇般在互联网中苦苦搜寻的时候，汤不热进入了我的视野。舶来品和国产品相差甚远，在使用方式和内容欣赏上都对我提出了巨大的挑战，在短暂的磨合期之后，我使用地得心应手。无论是夏天炎热燥烦的中午，还是冬天刺骨寒冷的深夜，有了它的陪伴，我总能找到内心的宁静。不久后某个不雅视频的泛滥让汤不热彻底地离开了我，又一次我不知所措，在无数个深夜里辗转反侧，难以入眠。不过这一次我选择了用知识武装自己，通过对BT下载的深入了解，我感受到了真正的自由，不受防火墙限制的那种自由，我不再计较一城一地的得失，因为信手拈来的都是我爱看的作品。

好了，下面言归正传，闲话少叙，敲黑板，正式进入BT知识的学习。

## BT介绍

BT全称BitTorrent，是一个分布式文件协议。区别于传统的HTTP/FTP下载，HTTP/FTP一般采用C/S架构，client需要从server下载文件数据，而BT是一种P2P协议，client在下载的同时，也承担着server的角色，把下载好的数据提供给其他client下载。这样文件源可以支持大量的用户进行下载，而只带来适当的负载的增长。

![HTTP/FTP下载 vs BT下载][image-2]

BT下载分为三步，下面我就这三个步骤详细讲解BT下载的细节。

1. 创建下载任务
2. 寻找peer
3. 下载文件

### 创建下载任务

在BT下载软件中，使用种子文件或者磁力链接创建下载任务。

#### 种子

种子就是后缀为torrent的文件，网上所说的`留图不留种，菊花万人捅`就是指的它。文件发布者在发布文件之前会根据发布文件生成一个种子文件，种子中包含着发布文件的详细信息。下载者拿到种子文件之后，就可以创建BT下载任务。用文本编辑器打开种子文件，你会发现它不是一个直接可读的文本文件。事实上，种子文件经过bencode编码，bencode编码很简单，有兴趣的同学可以通过[这篇博客][link-1]来详细了解bencode的详细细节。用torrent file editor打开种子文件，可以很直观地读取种子包含的下载信息。
 
![种子详情][image-3]

种子字段：

+ **info**：必须字段，一个描述torrent文件的字典，有两种可能形式，一种是没有目录结构的单一文件，一种是有目录结构的多文件。
+ **announce**：必须字段，tracker服务器的地址URL。
+ **announce-list**：，可选字段，tracker服务器列表，这是官方规范的一个扩展，向后兼容，可用来存储备用服务器列表。
+ **comment**：可选字段，一些备注信息。
+ **create date**：可选字段，torrent文件的创建时间，为Unix时间戳。
+ **created by**：可选字段，说明torrent文件是由哪个程序创建的。

info字段（又叫做种子的元数据 meta data）：

+ **piece length**：必须字段，每个piece的长度。
+ **pieces**：必须字段，20字节的SHA1散列值，每块(piece)一个，用于BT下载完成后校验是否出错。
+ **name**：必须字段，如果是单一文件，表示为文件名；如果是多文件，表示所在的文件夹名称。
+ **length**： 单一文件的情况下必须存在，多文件情况下没有此字段，表示文件的大小。
+ **files**：多文件情况下必须存在，单一文件下没有此字段，这是一个文件列表，列表中的每一项又包含两个必须字段：**path**表示文件路径，**length**表示文件大小。

以上只是最基本的字段，不同的种子制作工作都会添加不同的扩展字段。迅雷客户端打开种子文件后的任务创建列表就是展示的种子元数据。

![迅雷下载任务][image-4]

#### 磁力链接

类似于`magnet:?xt=urn:btih:AD28CACBAEA04BAD181685740D251D76E2EBA37A`的链接我们叫做磁力链接，它是通过种子文件内容的Hash结果生成一个纯文本的数字指纹。磁力链接中的四十位16进制字符串`AD28CACBAEA04BAD181685740D251D76E2EBA37A`称之为infohash，是由种子元数据（info字段）经过hash算法成成得到。很多BT工具都支持磁力链接创建下载任务，但实际上这些BT下载工具必须根据磁力链接生成种子文件，然后再根据种子文件去下载。每个种子文件可以生成一个唯一的磁力链接，相比于种子文件，磁力链接更小，很容易在互联网上传播，而且也更容易避开各个网站的审核，因此磁力链接被老司机们广泛接纳。

种子转换为磁力链接代码如下：

```python
# 利用bencode和hashlib
import bencode
import hashlib

torrent = open('/path/to/torrent', 'rb').read()
torrent_data= bencode.bdecode(torrent)
metadata= bencode.bencode(torrent_data['info'])
digest = hashlib.sha1(metadata).digest()
print 'magnet:?xt=urn:btih:%s' % digest.encode('hex')

# 利用libtorrent
import libtorrent
torrent_info = libtorrent.torrent_info('/path/to/torrent')
print 'magnet:?xt=urn:btih:%s' % torrent_info.info_hash()
```

在迅雷里面，如果先用种子创建一个下载任务，再用种子生成的磁力链创建一个下载任务，第二次创建的任务迅雷会提示任务已存在，说明无论是磁力链创建的下载还是种子创建的下载，迅雷会认为是同一种类型的下载，而且每个下载任务会以唯一的infohash做区分。

实际上，磁力链创建的下载任务第一步是根据磁力链中的infohash找到种子的元数据。这一步也是BT嗅探的关键一步，很多种子搜索引擎的核心也是围绕这个过程实现。各个BT下载软件一般有两种实现方式：建立种子仓库和DHT网络。以迅雷为例，迅雷有一个海量的种子仓库，并给客户端提供了根据infohash下载的接口，用wireshark抓包可以得到包文信息，只不过内容经过加密，有想法的同学可以洗洗睡了。第二种方式是通过DHT网络，我会在下面详细讲解DHT，只不过这种方式速度偏慢，并且无法保证成功。

### 寻找peer

要下载文件数据存储在peer节点中，因此需要找到当前正在做种的Peer节点。

#### Peer

简单理解，peer就是当前正在做种的BT客户端。举个例子，A和B都在下载Aquaman，而C和D正在为这个文件做种，那么B、C、D都是A的peer，它们都可以为A提供下载数据。如何快速地找到自己的peer，是评价一个下载客户端好坏的关键。目前有两种方式获取peer列表：使用tracker服务器和使用trackerless DHT网络两种方式。

#### tracker

种子文件中的announce和announce-list字段就是tracker的服务地址，tracker会维护种子正在活跃的peer列表。BT客户端在使用种子文件创建下载任务后，会把自己注册到tracker的活跃peer列表中，同时请求得到其他活跃的peer列表，之后BT客户端就可以和peer通信，下载文件数据。这种方式可以快速为BT客户端找到正在活跃的peer，但同时集中式的特点使得系统鲁棒性不好，一旦tracker服务器出现故障，整个下载也就不可持续了。

#### DHT

DHT全称叫分布式哈希表(Distributed Hash Table)，是一种分布式存储方法。在不需要服务器的情况下，每个客户端负责一个小范围的路由，并负责存储一小部分数据，从而实现整个DHT网络的寻址和存储。

### 下载文件

peer之间的通信协议又称为`peer wire protocal`，即peer连线协议，它是一个基于TCP协议的应用层协议。协议以piece为单位传输数据，peice的大小由种子的piece length字段定义，每个piece是否传输正确通过种子中的piece hash来验证。当整个文件下载完成之后，还需要校验整个文件正确性，一旦校验不通过，会导致部分piece重新下载，这也就是为什么有时候进度会长时间卡在99%的地方。

## 如何构建一个种子搜索引擎


[link-1]:	https://www.jianshu.com/p/c26de7a04c38

[image-1]:	https://upload-images.jianshu.io/upload_images/2027339-761cdf428a036cbe.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
[image-2]:	https://upload-images.jianshu.io/upload_images/2027339-76c35938d1875dae.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
[image-3]:	https://upload-images.jianshu.io/upload_images/2027339-f17268f29599083d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
[image-4]:	https://upload-images.jianshu.io/upload_images/2027339-a78bbc141ceac1f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240