## TCP粘包

------

### 粘包

1. 客户端粘包问题。客户端因为有Nagle优化算法，那么可客户端可能会把几个连续的send函数调用合并成一个包，称之为粘包，可以主动关闭nagle优化算法
2. 服务器粘包。即使客户端不粘包，服务端一定会粘包。因为recv之后需要进行处理，中间会有间隔，那么如果在下一次调用recv之前，多个包到达了 接收缓存区中，那么这些包就被组合成了一个包，即服务端粘包

### 解决

1. 客户端和服务端收发的数据包定义统一的格式：包头(固定长度)+包体
2. 在包头中记录整个包的长度
3. 先收包头的长度获得包头，然后根据包头的数据算出包体的长度，然后直接收取该长度的数据即可解决就=粘包问题
4. 为了防止内存字节对齐导致包的数据大小和实际大小不一致，必须使用1字节对齐。在定义该数据结构时，使用#pragma pack(1)指定对齐为1字节，# pragma pack()再次调取消该对齐指定。



### 缺包

如果自定义的包体的长度大于MTU，那么在传输过程中会可能会被分片，那么我们在一次recv调用时，只得到一部分的包体，这即称之为缺包。这仅仅是对于自定义接收发送方式下的概念。

### 接收状态

PKG_HEAD_INIT 初始状态，准备接收包头

PKG_HEAD_RECVING 包头不完整，继续接收

PKG_BODY_INIT 包头接收完，准备接收包体

PKG_BODY_RECVING 接收包体中，尚未接收完全

在接收数据过程中，将会一直处于这四种状态，不停的轮转