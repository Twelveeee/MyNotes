# RPC

RPC(Remote Procedure Call Protocol) 是指远程过程调用，也就是说两台服务器 A/B，一个应用部署在 A 服务器上，想要调用 B 服务器上应用提供的函数/方法，
由于不在一个内存空间，不能直接调用，需要通过网络来表达调用的语义和传达调用的数据。

一个完整的RPC架构里面包含了四个核心的组件，分别是Client，Client Stub，Server以及Server Stub，这个Stub可以理解为存根。

- 客户端(Client)，服务的调用方。
- 客户端存根(Client Stub)，存放服务端的地址消息，再将客户端的请求参数打包成网络消息，然后通过网络远程发送给服务方。
- 服务端(Server)，真正的服务提供者。
- 服务端存根(Server Stub)，接收客户端发送过来的消息，将消息解包，并调用本地的方法。

在远程调用时，我们需要执行的函数体是在远程的机器上的。这就带来了几个新问题：

**1.Call ID映射**。我们怎么告诉远程机器我们要调用这个函数，而不是Add或者FooBar呢？在本地调用中，函数体是直接通过函数指针来指定的，我们调用函数的时候，编译器就自动帮我们调用它相应的函数指针。
但是在远程调用中，函数指针是不行的，因为两个进程的地址空间是完全不一样的。所以，在RPC中，所有的函数都必须有自己的一个ID。这个ID在所有进程中都是唯一确定的。客户端在做远程过程调用时，必须附上这个ID。然后我们还需要在客户端和服务端分别维护一个 {函数 <--> Call ID} 的对应表。两者的表不一定需要完全相同，但相同的函数对应的Call ID必须相同。当客户端需要进行远程调用时，它就查一下这个表，找出相应的Call ID，然后把它传给服务端，服务端也通过查表，来确定客户端需要调用的函数，然后执行相应函数的代码。

**2.序列化和反序列化**。客户端怎么把参数值传给远程的函数呢？在本地调用中，我们只需要把参数压到栈里，然后让函数自己去栈里读就行。但是在远程过程调用时，客户端跟服务端是不同的进程，不能通过内存来传递参数。甚至有时候客户端和服务端使用的都不是同一种语言（比如服务端用C++，客户端用Java或者Python）。这时候就需要客户端把参数先转成一个字节流，传给服务端后，再把字节流转成自己能读取的格式。这个过程叫序列化和反序列化。同理，从服务端返回的值也需要序列化反序列化的过程。

**3.网络传输**。远程调用往往用在网络上，客户端和服务端是通过网络连接的。所有的数据都需要通过网络传输，因此就需要有一个网络传输层。网络传输层需要把Call ID和序列化后的参数字节流传给服务端，然后再把序列化后的调用结果传回客户端。只要能完成这两者的，都可以作为传输层使用。因此，它所使用的协议其实是不限的，能完成传输就行。尽管大部分RPC框架都使用TCP协议，但其实UDP也可以。

有了这三个机制，就能实现RPC了，具体过程如下：


```php
// Client端 
// int l_times_r = Call(ServerAddr, Multiply, lvalue, rvalue)
1. 将这个调用映射为Call ID。这里假设用最简单的字符串当Call ID的方法
2. 将Call ID，lvalue和rvalue序列化。可以直接将它们的值以二进制形式打包
3. 把2中得到的数据包发送给ServerAddr，这需要使用网络传输层
4. 等待服务器返回结果
5. 如果服务器调用成功，那么就将结果反序列化，并赋给l_times_r

// Server端
1. 在本地维护一个Call ID到函数指针的映射call_id_map，可以用std::map<std::string, std::function<>>
2. 等待请求
3. 得到一个请求后，将其数据包反序列化，得到Call ID
4. 通过在call_id_map中查找，得到相应的函数指针
5. 将lvalue和rvalue反序列化后，在本地调用Multiply函数，得到结果
6. 将结果序列化后通过网络返回给Client
```



**Client端** 

```php
call($service, $method, array $input = array(), array $options = array());
```

我们实现的call方法,需要指明service 的名字，function的名字，才能找到相应的service层的方法
在serviceConfig类里面找到baseUrl，和服务器集群配置

构造requestParams，比如timeout，http，header信息，proxy
目的地有了，参数有了，需要httpClient，发送到目的地，得到response，给Client端统一格式return

**Server端**
比如说c2citem模块，nginx指向webroot里的c2citem，里面只有一个index文件，指向bootstrap
bootstrap模块在启动的时候，自加载_initRoute，用的是Yaf_Route_Regex，正则路由，指向comrtoller里面一个api文件的callAction()方法 获取到`api.conf.php`的配置，设置缓存等等，到相应的service文件

```php
$route = new Yaf_Route_Regex('#api/' . Frame_AppEnv::getCurrApp() . '/([a-zA-Z0-9_]+)#', array('controller' => 'Api', 'action' => 'call'), array(1 => 'method'));
```

# 异步

串行

```php
   $rpc = new BBT_Service_Client();
    $input = array(
        'lookId' => 11048473649457,
    );
    $ret = $rpc->call('post', 'getLook', $input);
    if (!isset($ret['errno']) || 0 != $ret['errno']) {
       // 出错 return false  or throw Exception
    }
    $look = $ret['data'];

```



同步

```php
    $reqs = array(
        ['order', 'getSonOrderList', ['parentIdList' => $parentIdList]],
        ['order', 'getOrderItemByOrderParentIdArr', ['parentIdArr' => $parentIdList]],
        ['order', 'getParentOrderCount', ['userId' => $userId, 'state' => '']],
    );
    $retList = $client->multiCall($reqs);
    if ($retList[0]['errno'] != 0 || $retList[1]['errno'] != 0 || $retList[2]['errno'] != 0) {
        throw new BBT_Error(BBT_ErrorCodes::REQUEST_ERROR);
    }
```



异步

```php
    $input = array(
        'userIdList'   => $userIdList,
        'fields'       => array('multiLookId', 'userId'),
        'startTime'    => $nowTime - $multiLookPullRange,
        'num'          => $num,
        'auditStsList' => $auditStatusList,
    );
    $promise = $this->_client->asyncCall('post', 'getMultiLookByUserIdList', $input);
    $multiLookPromise = $promise->then(
        function ($ret) {
            // 以下是具体callback的实现
            // 由于php是单线程运行, 各callback间还是串行执行, 如果某个callback内有长耗时操作(比如说sleep), 将hang住所有其他异步请求的response处理
            // 因此在callback内, 最好不要有阻塞操作
            if ($ret['errno'] != 0) {
                throw new BBT_Error(BBT_ErrorCodes::REQUEST_ERROR);
            }
            $multiLookList = $ret['data']['multiLookList'];
            return $multiLookList;
        }
    );
    
    类似的
    $videoLookPromise = ...
    $coursePromise = ...
    
    // wait触发请求及对应回调并发执行
    $multiLookList = BBT_Service_Client::wait($multiLookPromise);
    $videoLookList = BBT_Service_Client::wait($videoLookPromise);
    $courseList = BBT_Service_Client::wait($coursePromise);
```



# 并发（秒杀）

抢购出现超卖的现象，问题如下
一个库存为1的商品出现了多个买家，

拍卖实现：
添加一个拍卖会场
用户加价，写入日志
拍卖结束，获取最后出价的用户
发货。

到结束时间，把超时的订单状态设置为超时，解决方案有3:
1.主动扫描，定时轮询：写个定时脚本，扫描未提交的订单，设置为超时。
2.被动提交：只有在查询订单的时候，判断订单的状态是否超时，如果超时再逻辑处理。
3.主动取消：设计一个消息环的时间结构，把所有需要时序操作的任务放上去，到点执行脚本。

# 负载均衡

云服务的负载均衡：

(Elastic Load Balancing) ELB弹性负载均衡 aws
(Classic Load Balancer) CLB传统型负载均衡 aliyun

**负载均衡的作用**
用于将网络流量分发到多个服务器上，以提高应用的响应速度和可用性。

高并发：负载均衡通过调整负载，尽力让应用集群中的节点工作量达到均匀，以此提高应用集群的并发处理能力（吞吐量）。
伸缩性：添加或减少服务器数量，然后由负载均衡进行分发控制。这使得应用集群具备伸缩性。
故障转移：负载均衡器可以监控候选服务器，当服务器不可用时，自动跳过，将请求分发给可用的服务器。这使得应用集群具备高可用的特性。
安全防护：有些负载均衡软件或硬件提供了安全性功能，如：黑白名单处理、防火墙，防 DDos 攻击等。



**负载均衡的类型**

1.硬件负载均衡
通过专门的硬件设备来实现负载均衡功能，F5和A10。
优点：功能强大，稳定性高，安全性高。
缺点：贵，扩展性差。

2.软件实现负载均衡
软件负载均衡，可以在普通的服务器上运行负载均衡软件，实现负载均衡功能。
目前常见的有 Nginx、HAproxy、LVS。其中的区别：

Nginx：七层负载均衡，支持 HTTP、E-mail 协议，同时也支持 4 层负载均衡；
HAproxy：支持七层规则的，性能也很不错;
LVS：运行在内核态，性能是软件负载均衡中最高的，严格来说工作在三层，所以更通用一些，适用各种应用服务。

优点：
易操作：无论是部署还是维护都相对比较简单；
便宜：只需要服务器的成本，软件是免费的；
灵活：4 层和 7 层负载均衡可以根据业务特点进行选择，方便进行扩展和定制功能。

## **负载均衡的算法**

轮询，随机，最少连接，一致性hash
1.轮询：按顺序把请求发到集群的各个节点上。
加权轮询：同上，但是有权重，性能更高的电脑会有更多的请求数。
2.随机：将请求随机分发到候选服务器。
加权随机：加入权重的随机算法。
3.最少连接：将请求发送到连接数最少的候选服务器上。
加权最少连接：基本同上
4.源地址hash算法：请求源ip的hash值对服务器数量取模，发送到这个服务器。
5.一致性hash：构建hash环，上面放服务器，根据源的hash值，顺时针找到最近的服务器。

**负载均衡的技术**

**1.DNS负载均衡**
一个域名解析到多个ip，每个ip都对应不同的ui服务器。
优点，实现简单，成本低。
缺点，服务器故障切换延迟大。dns缓存难以清除。

**2.HTTP负载均衡**
客户端发送请求到A服务器，A服务器返回真正的服务器地址，写到Http重定向相应中，由浏览器重新访问。
优点，简单方便。
缺点，性能差，如果A服务器宕机，就无法访问。

**3.反向代理**
反向代理（Reverse Proxy）方式是指以代理服务器来接受网络请求，然后将请求转发给内网中的服务器，并将从内网中的服务器上得到的结果返回给网络请求的客户端。反向代理负载均衡属于七层负载均衡。

反向代理服务的主流产品：Nginx、Apache。

正向代理：发生在客户端，是由用户主动发起的。客户端通过主动访问代理服务器，让代理服务器获得需要的外网数据，然后转发回客户端。
反向代理：发生在服务端，用户不知道代理的存在。

反向代理的优点：
多种负载均衡算法：支持多种负载均衡算法，以应对不同的场景需求。
可以监控服务器：基于 HTTP 协议，可以监控转发服务器的状态，如：系统负载、响应时间、是否可用、连接数、流量等，从而根据这些数据调整负载均衡的策略。

反向代理的缺点：
额外的转发开：反向代理的转发操作本身是有性能开销的，可能会包括，创建连接，等待连接响应，分析响应结果等操作。
增加系统复杂度：如果代理服务器宕机了，该怎么办。代理服务器页存在性能瓶颈。

**4.网络地址转发**
(Network Address Translation，简称 NAT)是指通过修改目的IP地址实现的负载均衡。NAT属于四层负载均衡。

工作原理
1.用户请求数据包到达 LB 后，LB 在操作系统内核进程获取网络数据包。
2.LB 器根据负载均衡算法得到 RIP，并将请求目的地址修改为 RIP，不需要经过用户进程处理。然后，将请求发送给 RS。
3.RS 处理完成请求后，将响应数据包返回到 LB。
4.LB 再将数据包源地址修改为自身的 IP 地址，发送给用户浏览器。

优点：在内核进程完成数据分发，比在应用层分发性能更好；
缺点：所有请求响应都需要经过负载均衡服务器，集群最大吞吐量受限于负载均衡服务器网卡带宽；

**5.隧道技术**
（ IP Tunneling，简称 TUN）负载均衡器把请求的报文通过 IP 隧道转发到真实的服务器。真实的服务器将响应处理后的数据直接返回给客户端。这样负载均衡器就只需处理请求报文。

**6.直接路由**
(Direct Routing，简称 DR)通过修改目的 MAC 地址实现负载均衡。DR 属于四层负载均衡。
负载均衡器收到用户数据包后，根据算法选出后端服务器，，把数据包的mac地址改为服务器的mac地址，发送出去，交换机识别mac地址，把数据交给服务器，服务器返回数据给客户端。
