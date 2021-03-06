本文重点：四、DNS解析流程 4、加上主机缓存后的DNS 解析流程


<br>


# 一、简介

### 1、DNS（Domain Name System，域名系统）

DNS（Domain Name System，域名系统）,它实质上就是个 域名 和 IP 相互映射的`联机分布式数据库`系统。

 - DNS作用：根据域名查出IP地址。
举例来说，如果你要访问一个地址为 hackr.jp的网页，首先要通过DNS查出它的IP地址是20X.189.105.112，然后你才能用该IP访问该网站。
![这X。图片描述](https://img-blog.csdn.net/20180820183346645)
 - DNS是分布式的：即单个计算机出故障，也不会妨碍整个DNS系统的正常工作。
 - 域名比起IP地址更加便于人们记忆，但是机器在处理IP数据报的时候使用的还是IP地址，因为其定长特性(IPv4为32位，IPv6为128位)便于机器处理，而域名是不定长的。

### 2、域名到IP地址的解析是由分布在因特网上的`许多域名服务器共同完成`的。

解析过程的要点如下：

（1）当某一个应用进程需要把域名解析为IP地址时，该应用进程就调用`解析程序`(resolver)，并成为DNS的一个客户，把待解析的域名放在DNS请求报文中，以`UDP用户数据报`方式(减少开销)发给本地域名服务器。

（2）本地服务器在查找到域名后，把对应的IP地址放在回答报文中返回。应用进程获得目的主机的IP地址后即可进行通信。

（3）若本地域名服务器不能回答该请求，则此域名服务器就暂时成为DNS中的另一名客户，并向其他域名服务器发出查询请求。这种过程直至找到能够回答该请求的域名服务器为止。


<br>


# 二、域名 分层结构    

### 1、因特网`采用层次树状结构的命名方法`

`之所以设计这样复杂的树形结构， 是为了防止名称冲突`。这样一棵树结构，当然可以存储在一台机器上，但现实世界中完整的域名非常多，并且每天都在新增、删除大量的域名，存在一台机器上，对单机器的存储性能就是不小的挑战。 另外，集中管理还有一个缺点就是管理不够灵活。 可以想象一下，每次新增、删除域名都需要向中央数据库申请是多么麻烦。 `所以现实中的DNS 都是分布式存储的`（eg:根域名服务器只存储与之相关联顶级域名）。

![这里写图片描述](https://img-blog.csdn.net/20180818174258994)



### 2、举例：
 下面用站长之家的域名举例:
 
![这里写图片描述](https://img-blog.csdn.net/20180818173932912)

 最上面的.是根域名, 接着是顶级域名com，再下来是站长之家域名chinaz 依次类推。 使用域名时，从下而上。 s.tool.chinaz.com. 就是一个完整的域名, www.chinaz.com. 也是。
 
`域名等级从左至右依次增高`：例如在www.chinaz.com中，www为三级域名，chinaz为二级域名，com为顶级域名。
有多个标号组成的完整域名总共不超过255个字符。

注：域名中的“点”和点分十进制IP地址中的“点”并无对应关系。


<br>


# 三、域名服务器
域名到IP地址的解析是由分布在因特网上的`许多域名服务器共同完成`的，现在依据域名服务器等级 从高到低 分为4类：

 - **根域名服务器(root name server)：** 最高层次,同时也是最重要的域名服务器。`任何一个根域名服务器都知道所有的顶级域名服务器的域名和IP地址`。
`本地域名服务器将 域名转换为IP地址 的过程中，只要自己无法转换，就要首先求助于根域名服务器。`假定所有的根域名服务器都挂了，那么整个互联网中的DNS系统就都挂了。
根域名服务器并不直接把待查询的域名直接转换成IP地址(根域名服务器也没有这种信息)，而是告诉本地域名服务器下一步应当找哪一个顶级域名服务器进行查询。
 - **顶级域名服务器(TLD服务器)：**`负责管理在该顶级域名服务器注册的所有二级域名`。当收到DNS查询请求时，就给出相应的回答(可能是最后结果，也可能是下一步应当找的域名服务器的IP地址)。
 - **权限域名服务器：** 从理论上讲，可以让每一级的域名都有一个相对应的域名服务器，使所有的域名服务器构成和 图6-1 相对应的“域名服务器树”的结构，但这样做会使域名服务器数量太多，使其运行效率降低，因此DNS采用`划分区`的方法来解决这个问题。
   `一个服务器所负责管辖的（或有权限的）范围叫做区`。`各单位根据自己情况来划分区的范围，每个区设置相应的 权限域名服务器，用来保存该区中所有主机的域名到IP地址的映射`。
 - **本地域名服务器(local name server)：**`当一个主机发出DNS查询请求时，这个查询请求报文就发给本地域名服务器`。每个互联网提供商ISP，或一个大学，甚至一个大学里的系，都可以拥有本地域名服务器。本地域名服务器离用户较近，一般不超过几个路由器的距离。当所要查询的主机也属于同一个本地ISP时，该本地域名服务器立即就能将所查询的主机名转换为IP地址，而不需要再访问其他域名服务器。


<br>

# 四、DNS解析流程

### 1、递归查询与迭代查询：

**a. 主机向本地域名服务器的查询一般都是采用递归查询**(recursive query)。如果主机所询问的本地域名服务器不知道被查询域名的IP地址时，`本地域名服务器就以DNS客户的身份向其他根域名服务器继续发送查询请求报文`(即替该主机继续查询)，而不是让该主机自己进行下一步的查询。

**b. 本地域名服务器向根域名服务器的查询通常是采用迭代查询**(iterative query)。当根域名服务器收到本地域名服务器发出的迭代查询请求报文时，要么给出所要查询的IP地址，要么告诉本地域名服务器下一步应当向哪一个域名服务器进行查询，然后`让本地域名服务器进行后续的查询`。逐步按照域树的路径向下走直到叶节点，得到了所要解析的域名的IP地址，然后把这个结果返回给发起查询的主机。当然本地域名服务器也可以采用递归查询，这取决于最初的查询请求报文的设置是要使用哪一种查询方式。 

<br>

### 2、解析流程:
![这里写图片描述](https://img-blog.csdn.net/20180818194200475)

假如客户端在浏览器的URL中输入y.abc.com，即想要解析获取y.abc.com的IP地址。（客户是第一次访问y.abc.com）

#### 如果 本地域名服务器 采用迭代查询：（图a）

（1）客户端向 本地域名服务器(递归查询) 发出解析y.abc.com域名的请求。
    本地域名服务器 查看本地缓存，是否有缓存过y.abc.com域名，如果有直接返回给客户端；如果没有执行下一步；
        
（2）本地域名服务器 采用迭代查询。它先向一个 根域名服务器 查询。

（3）根域名服务器 告诉 本地域名服务器，下一次应查询的 顶级域名服务器dns.com的IP地址。

（4）本地域名服务器 向 顶级域名服务器dns.com 进行查询。

（5）顶级域名服务器dns.com 告诉 本地域名服务器，下一步应查询的 权限服务器dns.abc.com 的IP地址。

（6）本地域名服务器 向 权限域名服务器dns.abc.com 进行查询。

（7）权限域名服务器dns.abc.com 告诉 本地域名服务器，所查询的主机的IP地址。

（8）本地域名服务器 最后把查询结果告诉m.xyz.com。

注：整个查询过程共用到了8个UDP报文。
<br>

#### 如果 本地服务器采用 递归查询：（图b）

这里，本地域名服务器 只需要向 根域名服务器查询一次，后面的查询都是在其他几个域名服务器之间进行的（步骤3~6）。只是在第7步，本地域名服务器 从 根域名服务器 得到了所需的IP地址。最后在步骤8，本地域名服务器 把查询得到的IP 告诉了客户端。整个查询过程也是使用了8个UDP报文。


<br>

### 3、高速缓存:
为了提高DNS查询效率，并减轻服务器的负荷和减少因特网上的DNS查询报文数量，在域名服务器中广泛使用了`高速缓存`，用来存放最近查询过的域名以及从何处获得域名映射信息的记录。

例如，在上面的查询过程中，如果在m.xyz.com的主机上不久前已经有用户查询过y.abc.com的IP地址，那么本地域名服务器就不必向根域名服务器重新查询y.abc.com的IP地址，而是`直接把高速缓存中存放的上次查询结果返回`(即y.abc.com的IP地址)给用户。

假定本地域名服务器的缓存中并没有y.abc.com的IP地址，而是存放着顶级域名服务器dns.com的IP地址，那么本地域名服务器也可以不向根域名服务器进行查询，而是直接向com顶级域名服务器 发送查询请求报文。这样能大大减轻根域名服务器的负荷。

由于名字到地址的绑定并不经常改变，为保持告诉缓存中的内容正确，域名服务器应`为每项内容设置计时器并删除超过合理时间的项`(例如每个项目两天)。当域名服务器已从缓存中删去某项信息后又被请求查询该项信息，就必须重新到授权管理该项的域名服务器绑定信息。当权限服务器回答一个查询请求时，在响应中都指明绑定有效存在的时间值。`增加此时间值可减少网络开销，而减少此时间值可提高域名解析的正确性。`


不仅在本地域名服务器中需要高速缓存，在主机中也需要。许多主机在启动时从本地服务器下载名字和地址的全部数据库，维护存放自己最近使用的域名的高速缓存，并且`只在从缓存中找不到名字时才使用域名服务器`。维护本地域名服务器数据库的主机应当定期地检查域名服务器以获取新的映射信息，而且主机必须从缓存中删除无效的项。由于域名改动并不频繁，大多数网点不需花精力就能维护数据库的一致性。

<br>

### 4、加上主机缓存后的DNS 解析流程:

如果还用上面的例子，且加上主机中的缓存，客户端在浏览器的URL中输入y.abc.com，即想要解析获取y.abc.com的IP地址。会发生一下动作：

（1）浏览器 会首先搜索`浏览器自身的DNS缓存`（缓存时间比较短，大概只有1分钟，且只能容纳1000条缓存），看自身的缓存中是否有y.abc.com 对应的条目，而且没有过期，如果有且没有过期则解析到此结束。

（2）如果浏览器自身的缓存里面没有找到对应的条目，那么浏览器会搜索`操作系统自身的DNS缓存`,如果找到且没有过期则停止搜索解析到此结束。

（3）如果在操作系统（eg:Windows系统）的DNS缓存中也没有找到，那么尝试读取`hosts文件`（位于C:\Windows\System32\drivers\etc），看看这里面有没有该域名对应的IP地址，如果有则解析成功。

（4）如果在hosts文件中也没有找到对应的条目，浏览器就会发起一个DNS的系统调用，就会向 `本地域名服务器` 发起域名解析请求

（5）之后的操作就一样了，同上···· 即：上面 本地域名服务器 递归查询或迭代查询 的过程。

<br>

# 五、DNS负载均衡

DNS还有负载均衡的作用，现在很多网站都有多个服务器，当一个网站访问量过大的时候，如果所有请求都请求在同一个服务器上，可能服务器就会崩掉，这时候就用到了DNS负载均衡技术，当一个网站有多个服务器地址时，在应答DNS查询的时候，DNS服务器会对每个查询返回不同的解析结果，也就是返回不同的IP地址，从而把访问引导到不同的服务器上去，来达到负载均衡的目的。例如可以根据每台机器的负载量，或者该机器距离用户的地理位置距离等等条件。


