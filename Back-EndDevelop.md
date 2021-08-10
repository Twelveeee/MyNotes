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

