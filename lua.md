# lua语法笔记



### 内置函数

```lua
--table获取值
--用rawget(t,i ） 会对表t 进行原始（ raw ） 的访问，即在不考虑元表的情况下对表进行简单的访问。
function rawget(table, index)
    
--table设置值
function rawset(table, index, value) 

--设置table的元表
function setmetatable(table, metatable)
```





：会携带self参数

__call元方法： 当table名字做为函数名字的形式被调用的时候，会调用__call函数

__index方法除了可以是一个表，还可以是一个函数，如果是一个函数，__index方法被调用时将返回该函数的返回值。


string.byte(arg[,int]) 在 Lua 中 byte 函数用于转换字符为整数值(可以指定某个字符，默认第一个字符)，并返回转换后的数字。

1 使用function声明的函数为全局函数，在被引用时可以不会因为声明的顺序而找不到 
2 使用local function声明的函数为局部函数，在引用的时候必须要在声明的函数后面


tbl.login 等于 tbl["login"]


string.find用于找出字母在字符串中的位置。


当函数参数是一个单独的字符串和表 可以省略函数括号



### **string.pack**



<: 设为小端编码
>: 设为大端编码
>=: 大小端遵循本地设置
>![n]: 将最大对齐数设为 n （默认遵循本地对齐设置）
>b: 一个有符号字节 (char)
>B: 一个无符号字节 (char)
>h: 一个有符号 short （本地大小）
>H: 一个无符号 short （本地大小）
>l: 一个有符号 long （本地大小）
>L: 一个无符号 long （本地大小）
>j: 一个 lua_Integer
>J: 一个 lua_Unsigned
>T: 一个 size_t （本地大小）
>i[n]: 一个 n 字节长（默认为本地大小）的有符号 int
>I[n]: 一个 n 字节长（默认为本地大小）的无符号 int
>f: 一个 float （本地大小）
>d: 一个 double （本地大小）
>n: 一个 lua_Number
>cn: n字节固定长度的字符串
>z: 零结尾的字符串
>s[n]: 长度加内容的字符串，其长度编码为一个 n 字节（默认是个 size_t） 长的无符号整数。
>x: 一个字节的填充
>Xop: 按选项 op 的方式对齐（忽略它的其它方面）的一个空条目
>' ': （空格）忽略
>————————————————



### 魔法字符

%a	字母
%c	控制字符
%d	数字
%l	小写字母
%p	标点符号
%s	空白字符
%u	大写字母
%w	字母或数字
%x	十六进制字符
%z	内部表示为0的字母

大写表示补集

+	重复1次或多次 尽可能多	
*	重复0次或多次 尽可能多
-	重复0此或多次 优先0个 迫不得已才多个
	？	可选部分（重复0次或1次）

使用 () 可以将匹配的值单独捕获：
使用[]可以创建字符集 开头加^表示补集

^表示开头 $表示结尾	
%b表示成对匹配 %bxy表示 匹配x开头 y结尾的字符串

[[]]框起来的字符串 无须转义


string.gsub( s, pattern, rep1[, n] );

第三个三次表示符合的替换成rep1

rep1	替换串，将s中包含的pattern替换成rep1
n	替换次数，从左到右开始，省略表示全替换





### C库

导入xx.dll，要添加package.cpath





#### 协程



1.coroutine.create(function()）

创建是挂起状态



2.coroutine.status(co) 

查看协程状态



3.coroutine.resume(co)

启动或者再次启动

返回值第一个是boolean ，true表示没有错误，接着的返回值是yield的参数



第一次 resume的参数对应create的参数

yield之后调用resume的参数对应yield的返回值



4.coroutine.yield()

返回值是对应的唤醒他的resume的参数







### **self和assert**



assert是断言，当v为true，返回v，不为true，error，显示message

```lua
function assert(v, message)
```



-------------------------------

如果函数内使用self

当用 “ . ” 调用函数时，第一个参数是self，所以传入第一个参数可以改变self的值

当用 “ : ” 调用函数时，self隐式传递，即所在table里面的属性，



assert断言一个函数时，	

```
local f = assert(REQUEST[name])
local r = f(args)

function REQUEST:get()
	print("get", self.what)
	local r = skynet.call("SIMPLEDB", "lua", "get", self.what)
	return { result = r }
end

```



在REQUEST[name]函数中并且没有接收函数，且定义的时候是：，函数里面也使用了self，

这个时候f(args)这个语句传递的参数是self，所以推测assert返回的函数调用是 . 的模式，



### pcall

```lua
function pcall(f, arg1, ...)
```

没有出错pcall返回一个布尔值，