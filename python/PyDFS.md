功能需求：

* 文件的上传、下载、删除

其他需求：

* 文件的元数据本地缓存

* 多用户支持，可以同时读。有用户在读时不能写，写互斥。

系统架构：





# Solution

### server向client询问是否覆盖文件

可以借助回调函数，以下是一个简单的例子：

```python
#server.py
import rpyc
from rpyc import ThreadedServer


class CallBackService(rpyc.Service):
    class exposed_CB():
        def exposed_cb(self,func):
            a = func()
            print(a)


if __name__ == '__main__':
    t = ThreadedServer(CallBackService, port=9000)
    t.start()
   
#client.py
import rpyc


def func():
    return "a"


conn = rpyc.connect('localhost', port=9000)
cb = conn.root.CB()
cb.cb(func)
```

server会调用client端的func函数，而func的执行结果会返回到server

