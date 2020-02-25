## FTP协议

------

ftp包含两个数据通道：命令通道和数据通道(文件目录信息也通过数据通道进行传输)，在一次链接过程中，数据通道可能会发生变化，即数据传输通道在每一次文件上传和下载后都会关闭，直到下一次再打开。

### FTP工作模式

1. 主动模式 PORT

   客户端首先和服务器的TCP 21端口建立连接，通过这个通道发送命令，客户端需要接收数据的时候在这个通道上发送PORT命令, PORT命令包含了客户端用什么端口接收数据

2. 被动模式 PASV

   FTP服务器收到Pasv命令后，随机打开一个临时端口用于传送数据

### 常用命令和状态码

1. 连接成功

   220 Welcome to FtpServer\r\n

2. USER 用户登录

   USER root\r\n

   230 Login successful.\r\n

3. PWD 获取当前目录

   PWD  \r\n

   257 "/" is current directory.（引号中的内容即为路径, 客户端需要自行解析）

4. CWD 进入目录

   CWD test\r\n

   250 Directory success changed.

5. CDUP 返回上层目录

   CDUP\r\n

   250 Directory success changed.

6. PORT 客户端发送数据传送地址和端口

   PORT 127,0,0,1,70,96\r\n

   200 PORT command successful.\r\n

7. 端口计算方法

   PORT n1,n2,n3,n4,n5,n6\r\n(前四个数字组成IP点分十进制，后两个为端口号)

   port = n5*256 + n6

8. LIST 获取目录(数据通道连接，发送目录给客户端)

   LIST\r\n

   150　Here comes the directory listing.\r\n

   450　file open failed.

   linux返回的目录文件信息（文件类型和权限  子目录个数  所有者  用户组  文件大小  月  日  时间  文件名）

   -rwxrwxrwx 1 root group 64463 Mar 14 09:53 101.jpg\r\n   （相当于linux的shell 命令：ls -l）

   226 Transfer complete\r\n   服务端主动关闭数据通道

9. RETR 下载文件(数据通道连接传送文件数据给客户端)

    RETR filepath\r\n

    150 Transfer start.\r\n

    450　file open failed.

    226 Transfer complete\r\n   服务端终端关闭数据通道

10. STOR 上传文件(打开数据通道， 服务端读取上传的文件，客户端发送结束会主动关闭数据通道)

     STOR filepath\r\n

     125 file OK.\r\n

     226 Transfer complete\r\n  客户端主动关闭数据通道