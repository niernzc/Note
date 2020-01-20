## RPyC简介

一个简单的例子：

client.py

```python
import rpyc

# run a client which connect to the server
conn = rpyc.connect("localhost", port=9999)
cResult = conn.root.test(11)
conn.close()
print(cResult)
```

server.py

```python
from rpyc import Service
from rpyc.utils.server import ThreadedServer


class TestService(Service):
    def exposed_test(self, num):
        return 1 + num


sr = ThreadedServer(TestService, port=9999, auto_register=False)
sr.start()
```



# tutorial

## Part1

```shell
git clone https://github.com/tomerfiliba/rpyc.git
```

开启一个经典模式下的server

```shell
nzc@ubuntu:~/python/rpyc$ python bin/rpyc_classic.py
INFO:SLAVE/18812:server started on [127.0.0.1]:18812
```

SLAVE为服务类型，同时信息还有服务的地址和端口

```python
>>> import rpyc
>>> conn = rpyc.classic.connect("localhost")
>>> print >> conn.modules.sys.stdout,"hello world"
```

第三行是python2 print 标准输出的重定向

在server得到输出

```python
INFO:SLAVE/18812:accepted ('127.0.0.1', 41632) with fd 4
INFO:SLAVE/18812:welcome ('127.0.0.1', 41632)
hello world
```

在python3中的重定向

```python3
>>> print("Hello World!", file=conn.modules.sys.stdout)
```

### builtin Namespace

传统连接的builtin属性将暴露服务器python环境中的所有内建函数

```python
>>> f = conn.builtin.open('/home/nzc/.ssh/id_rsa')
>>> f.read()
'-----BEGIN RSA PRIVATE KEY-----\n-----END RSA PRIVATE KEY-----\n'
```

### `eval`和`execute`方法

classic connections 的这两个方法允许直接计算任意的表达式或者执行任意的指令

```python
>>> conn.execute('import math')
>>> conn.eval('2*math.pi')
6.283185307179586
```

### `teleport`方法

将函数发送到另一端并在那执行

```python
>>> def square(x):
...     return x ** 2
... 
>>> fn = conn.teleport(square)
>>> fn(2)
4
>>> conn.eval('square(2)')
4
>>> conn.namespace['square']
<function square at 0x7f4cf6576250>
>>> conn.namespace['square'] is fn
True
```

发送过去的函数在另一端的命名空间被定义好了

## Part2

```python
>>> type(conn.modules.sys)
<netref class 'rpyc.core.netref.builtins.module'>
>>> type(conn.modules.sys.path)
<netref class 'rpyc.core.netref.builtins.list'>
```

**netref** (network references)，一个透明的对象代理（transparent object proxy），他们将在他们之上执行的所有操作都委托给对应的远端对象。Netref本身并不是list或func，但它看起来像是

```python
>>> isinstance(conn.modules.sys.path,list)
True
```

## Part3: Services and New Style RPyC

自3.00起，RPyC的变成模型开始基于`service`。在classic mode中，client基本拥有对于server的完全控制，因此我们将这种服务称为slave。新的模型是面向服务的：服务提供了向另一方公开一组定义良好的功能的方法（这使得RPyC成为一个通用的RPC平台）。

服务的样板：

```python
import rpyc

class MyService(rpyc.Service):
    def on_connect(self, conn):
        # code that runs when a connection is created
        # (to init the service, if needed)
        pass

    def on_disconnect(self, conn):
        # code that runs after the connection has already closed
        # (to finalize the service, if needed)
        pass

    def exposed_get_answer(self): # this is an exposed method
        return 42

    exposed_the_real_answer_though = 43     # an exposed attribute

    def get_question(self):  # while this method is not exposed
        return "what is the airspeed velocity of an unladen swallow?"
```

可以自由地定义类，同时通过给方法名加上前缀`exposed_`来将方法/属性暴露给另一方，否则这些方法属性只有局部可以访问，在上例中，客户端可以调用`get_answer`但不能调用`get_question`

将服务暴露给外界：

```python
if __name__ == "__main__":
    from rpyc.utils.server import ThreadedServer
    t = ThreadedServer(MyService, port=18861)
    t.start()
```

对于远程方，服务作为连接的**根对象**公开：

```python
>>> import rpyc
>>> c = rpyc.connect("localhost", 18861)
>>> c.root
<__main__.MyService object at 0x834e1ac>
```

这个“根对象”是对远程进程中的服务实例的一个引用，可用于访问和调用暴露的属性和方法

```python
>>> c.root.get_answer()
42
>>> c.root.the_real_answer_though
43
```

### 访问策略

默认情况下只有带有`exposed_`前缀的属性和方法是可见的，这意味着默认情况下内建对象（如元组、字典）的属性是无法访问的。如果需要，可以在创建服务时传递适当的参数来进行相关的配置：

```python
from rpyc.utils.server import ThreadedServer
server = ThreadedServer(MyService, port=18861, protocol_config={
    'allow_public_attrs': True,
})
server.start()
```

### 共享的服务实例

我们将**类** MyService传递给server，使得每个到来的连接都能使用自己的独立的MyServer实例作为根对象

如果传递的是一个实例，那么所有连接都会使用同意个实例作为他们共享的根对象:

```python
t = ThreadedServer(MyService(), port=18861)#注意括号
```

如果你想要传递一个完全构造的服务实例，同时又希望每个连接的根对象分开，可以如下操作：

```python
from rpyc.utils.helpers import classpartial

service = classpartial(MyService, 1, 2, pi=3)
t = ThreadedServer(service, port=18861)
```

> on_connect` and `on_disconnect, Passing instances and classpartial 都是4.0以后加入的

### Registry Server

每个服务都有一个名字，通常是类名减去`“service”`后缀。服务名不区分大小写，如果想要定义一个服务名，可以通过设置`ALIASES`列表，该列表第一个元素是“正名”，其他的是“别名”

```python
class SomeOtherService(rpyc.Service):
    ALIASES = ["floop", "bloop"]
    ...
```

```python
>>> c.root.get_service_name()
'MY'
>>> c.root.get_service_aliases()
('MY',)
```

服务的名字用于**服务注册**：server会将它的details广播给附近的registry server以被发现

`bin/rpyc_registry.py`便是一个registry server，它会监听广播UDP socket，并回答关于哪个服务正在何处运行的查询。如果有一个registry server在网络上进行广播，而且其他server都配置为自动注册，则客户端可以自动地发现这些服务：

```python
>>> rpyc.discover("MY")
(('192.168.1.101', 18861),)
```

以及如果不在意具体连接到哪个服务，可以通过服务来进行连接：

```python
>>> c2 = rpyc.connect_by_service("MY")
>>> c2.root.get_answer()
42
```

<font color=red>(服务发现这一块，本地起两个server线程和一个registry server线程（来自github的registry server），未能找到相关服务)</font>





### 回调函数

所谓回调函数是指将一个函数像值一样进行传递啊，在c和c++中可以使用函数指针，在python中：

```python
>>> def f(x):
...     return x**2
...
>>> map(f, range(10))   # f is passed as an argument to map
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```

在rpyc的场景中：

```python
>>> import rpyc
>>> c = rpyc.classic.connect("localhost")
>>> rlist = c.modules.__builtin__.range(10) # this is a remote list
>>> rlist
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>>
>>> def f(x):
...     return x**3
...
>>> c.modules.__builtin__.map(f, rlist)  # calling the remote map with the local function f as an argument
[0, 1, 8, 27, 64, 125, 216, 343, 512, 729]
>>>

# and to better understand the previous example
>>> def g(x):
...     print "hi, this is g, executing locally", x
...     return x**3
...
>>> c.modules.__builtin__.map(g, rlist)
hi, this is g, executing locally 0
hi, this is g, executing locally 1
hi, this is g, executing locally 2
hi, this is g, executing locally 3
hi, this is g, executing locally 4
hi, this is g, executing locally 5
hi, this is g, executing locally 6
hi, this is g, executing locally 7
hi, this is g, executing locally 8
hi, this is g, executing locally 9
[0, 1, 8, 27, 64, 125, 216, 343, 512, 729]
```

这说明**RPyC**是对称的：

![../_images/symmetry.png](https://rpyc.readthedocs.io/en/latest/_images/symmetry.png)

## Part5 ：异步操作和事件

到目前的所有代码都是同步操作的，而异步操作时RPyC的一个key feature。在同步操作中，调用函数后会等待直到函数返回一个结果，而在异步操作中，你会得到一个`AsyncResult`，（又或者叫做`future`或`promise`），他们最终将装载结果。

异步请求的操作顺序不定

为了将远程函数的调用改成异步的，只需要使用`async_`对调用进行封装。这会创建一个返回`AsyncResult`对象的wrapper function（包裹函数）。该对象拥有如下属性和方法：

- `ready` - 表明结果是否已经到达
- `error` - 表明结果是一个值还是一个错误
- `expired` - 表明结果是否已超时，（需要设置set_expire，否则不会超时）
- `value` - 包含在`AsyncResult`中的值，如果尚未ready，则该对属性的访问会被阻塞，如果返回的是一个异常，则对其的访问会引发该异常，如果超时，则会引发一个异常，否则，将会返回该值
- `wait()` - 等待该结果返回，或者超时
- `add_callback(func)` - 添加一个回调函数，其将在结果返回时调用
- `set_expiry(seconds)` - 设置超时时间，默认不设置

示例：

```python
>>> import rpyc
>>> c=rpyc.classic.connect("localhost")
>>> c.modules.time.sleep
<built-in function sleep>
>>> c.modules.time.sleep(2) # i block for two seconds, until the call returns

 # wrap the remote function with async_(), which turns the invocation asynchronous
>>> asleep = rpyc.async_(c.modules.time.sleep)
>>> asleep
async_(<built-in function sleep>)

# invoking async functions yields an AsyncResult rather than a value
>>> res = asleep(15)
>>> res
<AsyncResult object (pending) at 0x0842c6bc>
>>> res.ready
False
>>> res.ready
False

# ... after 15 seconds...
>>> res.ready
True
>>> print res.value
None
>>> res
<AsyncResult object (ready) at 0x0842c6bc>
```

```python
>>> aint = rpyc.async_(c.modules.__builtin__.int)  # async wrapper for the remote type int

# a valid call
>>> x = aint("8")
>>> x
<AsyncResult object (pending) at 0x0844992c>
>>> x.ready
True
>>> x.error
False
>>> x.value
8

# and now with an exception
>>> x = aint("this is not a valid number")
>>> x
<AsyncResult object (pending) at 0x0847cb0c>
>>> x.ready
True
>>> x.error
True
>>> x.value #
Traceback (most recent call last):
...
  File "/home/tomer/workspace/rpyc/core/async_.py", line 102, in value
    raise self._obj
ValueError: invalid literal for int() with base 10: 'this is not a valid number'
```

### 事件

