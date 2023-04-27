



# skynet笔记



配置文件路径和在哪里执行有关



### skynet api



skynet.register_protocol 自定义消息类型 pack unpack 和dispatch

skynet.dispatch 通常是接受lua类型的消息

skynet.trash 释放c的内存

skynet.ignoreret() 使skynet.ret失效



### cluster

cluster.query(node, name) 在远程节点上查询一个名字对应的地址。
cluster.proxy(node, address) 为远程节点上的服务创建一个本地代理服务。
cluster.call(node, address, ...) 向一个节点上的一个服务提起一个请求，等待回应。
cluster.send(node, address, ...) 向一个节点上的一个服务推送一条消息。



cluster.open(port)

port是个字符串 起个名字监听这个服务

### snax





### datacenter 

类似一个全网络共享的注册表。它是一个树结构，
任何人都可以向其中写入一些合法的 lua 数据，其它服务可以从中取出来。
所以你可以把一些需要跨节点访问的服务，自己把其地址记在 datacenter 中，需要的人可以读出。

local datacenter = require "skynet.datacenter"



datacenter.set(key1, key2, ... , value)
可以向 key1.key2 设置一个值 value 。
这个 api 至少需要两个参数，没有特别限制树结构的层级数。

datacenter.get(key1, key2, ...) 从 key1.key2 读一个值。这个 api 至少需要一个参数，
如果传入多个参数，则用来读出树的一个分支。

datacenter.wait(key1, key2, ...) 同 get 方法，但如果读取的分支为 nil 时，
这个函数会阻塞，直到有人更新这个分支才返回。
当读写次序不确定，但你需要读到其它地方写入的数据后再做后续事情时，
用它比循环尝试读取要好的多。wait 必须作用于一个叶节点，不能等待一个分支。





### gate



**gateserver**

 来自于 require "snax.gateserver"

启动函数 gateserver.start(handler)

handle是一个函数表，定义一些相当于事件函数

如connect，disconnect，message，在创建连接，断开连接，发送消息的时候调用

客户端发送的消息要符合头部两个字节表示大小，两个字节的值表示后面数据的字节数





**snax.gateserver**

负责管理socket连接



入口为CMD.open，通过lua消息触发，参数为config

通过socketdriver建立连接，存储fd



```lua
		socketdriver.start(socket)
		--触发上面说的handle里面的函数
		if handler.open then
			return handler.open(source, conf)
		end
```



coroutine.running()   返回正在跑的 coroutine



```lua
MSG.more = dispatch_queue

local function dispatch_queue()
	local fd, msg, sz = netpack.pop(queue)
    
	if fd then
        -- 这里自己调用自己，相当于死循环
		skynet.fork(dispatch_queue)
		dispatch_msg(fd, msg, sz)
        
        -- 这里清空queue里面的message
		for fd, msg, sz in netpack.pop, queue do
			dispatch_msg(fd, msg, sz)
		end
	end
end
--从netpack中取出消息，netpack模块是c编写，
```





gateserver启动流程

1.入口为gateserver.start，由gate.lua调用

2.启动socket为start里面的CMD.open，使用socketdriver建立连接

参数是带有host和port的config，不带host默认0.0.0.0 



start函数定义了一些函数，用于处理skynet消息



gateserver有两大类命令，一是CMD，二是MSG

CMD.open用于打开连接，是服务端的监听触发**handler.open**

CMD.close关闭连接



MSG.open接受客户端连接，超过最大连接会shutdown，触发**handler.connect**

MSG.close关闭客户端连接和服务端的监听连接，触发handler.disconnect

**MSG.data**绑定dispatch_msg，dispatch_msg调用handler.message，handler.message是接收消息的方法

**MSG.more**相当于处理多个msg，msg存储在queue里面，实际还是调用dispatch_msg



其他MSG命令都触发了同名的handler函数，相当于回调



CMD命令在接受lua类型消息的时候调用，在init方法里面skynet.dispatch接收

MSG命令在接受socket类型消息的时候调用，在skynet.register_protocol里面



最后start方法调用skynet.start(init)，创建了服务



### gate/watchdog/agent



gate.lua 是 gateserver 是实现，主要是编写了handler传入gateserver中



watchdog创建

gate创建





watchdog cmd: start



gate.lua [socket type]  

dispatch参数：（q=nil, type=init, 1 ,0.0.0.0, 8080）

第一次调用MSG.init(id, addr, port)



gate.lua [socket type]  

dispatch参数：（q=nil, type=init, 1 ,start, 0）

第二次 调用MSG.init(id, addr, port)





**客户端进行连接**

```
gate.lua [socket type] 

dispatch参数：（q=nil, type=open, 2, 127.0.0.1:50634）

调用MSG.open(fd, msg)
```



MSG.open触发gate.lua的handler.connect(fd,addr)，看上面，msg内容就是ip地址，

然后创建了

```lua
connection[fd] = { fd,ip} --  判断连接存在的表
```

然后向watchdog 发送**socket** **open**消息







```
watchdog  cmd:  socket open
```

watchdog在SOCKET.open创建agent

```lua
agent[fd] = skynet.newservice("agent")
```

然后向agent发送**start**消息



```lua
agent cmd: start
```

agent在CMD.start对参数client, gate, watchdog进行保持，然后调用skynet.fork 每5秒发送一次心跳heartbeat，貌似是使用sproto协议。然后向gate发送**forward**消息



gate在CMD.forward先判断connection[fd]是否存在，然后调用unforward，把connection的agent和client赋值nil，然后将agent和client（即fd）赋值成参数里的值。然后调用gateserver.openclient(fd)



gateserver.openclient(fd)进行socketdriver.start(fd)，正式建立连接



```
gate.lua [socket type] 

dispatch参数：（q=nil, type=init, 2, start, 0）

调用MSG.init(id, addr, port)
```







##发一条消息

gate.lua [socket type] 

dispatch参数：（q=userdata, type=data, 3，userdata，1） 两个userData不一样

调用MSG.data()  即dispatch_msg(fd, msg, sz)

因为我修改了客户端发送一条，所以旧fd断开连接了，这里fd=3



##发两条消息

gate.lua [socket type] 

dispatch参数：（q=userdata, type=more）

调用MSG.more()



gate.lua [socket type] 

dispatch参数：（q=userdata, type=close, 2）

调用MSG.close(fd)



watchdog  cmd:  socket close

agent cmd disconnect



![1681787877525](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1681787877525.png)





发送消息在agent里面的send_package方法

里面require了skynet.socket，调用socket.



### 多次重启socket接收不到消息

```lua
--执行
killall -9 skynet
```



### **socket**

socket.start之后不能skynet.exit，不然服务直接结束了

有返回显示的是telnet



### **sproto协议**

skynet examples中agent使用是sproto协议





.开头是定义自定义数据结构



**luasocket库要在send的数据最后加\n**

client:**receive(**[pattern [, prefix]]**)**

- '`*l`': reads a line of text from the socket. The line is terminated by a LF character (ASCII 10), optionally preceded by a CR character (ASCII 13). The CR and LF characters are not included in the returned line. In fact, *all* CR characters are ignored by the pattern. This is the default pattern;

*l：从套接字中读取一行文本。 该行以一个 LF 字符 (ASCII 10) 结束，前面可以选择一个 CR 字符 (ASCII 13)。 返回行中不包含 CR 和 LF 字符。 事实上，*所有* CR 字符都被模式忽略了。 这是默认模式；



设置数字参数，不是一次读小于这个字节数，而是等缓冲区够多少一起读



**ps：离谱**





#### sproto rpc

sproto 提供的第一种 rpc 模式封装了上面的流程。

你需要定义一个叫做 package 的消息类型，里面包含 type 和 session 两项。

对于每个包，都以这个 package 开头，后面接上 (padding）消息体。最后连在一起，用 sproto 自带的 0-pack 方式压缩。

你可以用 sproto:host 这个 api 生成一个消息分发器 host ，用来处理上面这种 rpc 消息。

默认每个 rpc 处理端都有处理请求和处理回应的能力。也就是每个 rpc 端都同时可以做服务器和客户端。所以 host:dispatch 这个 api 可以处理消息包，返回它是请求还是回应，以及具体的内容。

如果 host 要对外发送请求，它可以用 host:attach 生成一个打包函数。这个生成的函数可以将 type session content 三者打包成一个串，这个串可以被对方的 host:dispatch 正确处理。

具体的说明，我放在了 [skynet 的 wiki 页](https://github.com/cloudwu/skynet/wiki/Sproto) 上。









**client_proto，server_proto**为你自定义的协议对象，可以由sproto.parse或new生成



```lua
--sp协议里面名为packageName的类型，得包含type和session
local client = client_proto:host("package")
--server_proto是客户端要发送的sproto对象，函数返回一个发送器sender
-- 发送器
local client_request = client:attach(server_proto)

--调用sender可以将你的obj封装成协议里面名为name类型
-- sender(name,request,session)，这里命名为client_request，区分客户端服务端
local req = client_request("foobar", { what = "foo" }, 1)

-- 通过luasocket发送req
socket.send(req)

--服务端接收,sp:host这里的sp就是代表你接收的协议了，不是发送的，发送的在attach里面用参数传
local server = server_proto:host("package")
-- 如果是解析request，会有四个参数,req为字节
-- 这个时候type == "request"
local type, name, request, response = server:dispatch(req)

--第四个参数response是function类型，也是个发送器，相当于做出回应，会自动填充session为你sender时的session，参数得符合request所在协议里面的response结构
local resp = response(ok = true)

--通过socket将resp发给客户端，打包成前两个字节代表长度，最后的”\n“，luasocket客户端只能readline，skynet的socket接收就不用
resp = string.pack(">s2",resp).."\n"
socket.write(resp)

--客户端接收，如果是request，则同上，如果是服务端调用response返回的数据
-- dispatch会有不同的返回值
-- 这个时候type == "RESPONSE"
local type, session, response = client:dispatch(resp)

```

