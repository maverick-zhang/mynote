## FTPServer

------

![类图](../static/类图.png)

#### XFTPFactory

​	工厂类，创建CMD指令对象，并把指令对象注册到XFTPServerCMD中

#### XFTPServerCMD

​	接收客户端的命令并分发处理对象，接收处理对象的注册

#### XTask

​	接口类，

#### XFTPTask

​	继承用的类，封装一些task共用的代码