## epoll

------

- epoll时linux系统提供的io复用技术，是基于poll进行的优化，能够支持高并发
- epoll在面对高并发连接时，但是只处理活跃的连接，是一种事件驱动的io模型



### 原理

epoll主要由三个函数组成([源码实现参考](https://github.com/wangbojing/NtyTcp/))

epoll_create(), epoll_ctl(), epoll_wait()

1. int epoll_create(int size)

   创建一个eventpoll对象，返回该对象的描述符，参数size需要大于0

   该函数首先分配一块内存给eventpoll这个结构对象，然后把结构对象的成员进行初始化

   eventpoll的成员

   1. ​	一颗红黑树的根节点指针rbr
   2. 一个双向链表的表头指针rdlist

2. int epoll_ctl(int efpd, int op, int sockid, struct epoll_event * event)

   把一个客户端连接的socket放入(或从中删除，修改)到eventpoll中，即把其作为节点添加到eventpoll中的红黑树上

   efpd为epoll_create返回的对象描述符

   op为动作，EPOLL_CTL_ADD, EPOLL_CTL_DEL, EPOLL_CTL_MOD，添加、删除和修改

   红黑树上的节点

   event为事件信息

   

   对于ADD操作，首先根据socketid查找红黑树上是否有这个节点，如果没有则生成一个epitem结构对象，这个结构对象就是红黑树的节点(同时也是双向链表的节点)，成员包括

   rbn， 一个结构， 包括三个指针，左子树，右子树和父节点以及节点的颜色

   rdlink 一个结构，包含两个指针，即链表的next和prev

   rdy 一个标记，表示该节点是否在双向链表上

   sockfd 连接的套接字

   event 事件

3. int epoll_wait(int epfd, struct epoll_evnet * events, int maxevents, int timeout)

   阻塞一小段时间并等待事件的发生，返回事件的集合 。即遍历双向链表，把链表的节点数据进行拷贝，然后从双向链表中移除

   events是事件数组，长度为maxevents，表示最多收集到的准备好的事件

   timeout表示阻塞的时间

   
   
   4.操作系统往双向链表添加节点
   
   操作系统调用epoll_event_callback
   
   四种情况
   
   - 客户端完成三次握手 accept
   - 客户端关闭连接 close
   - 客户端发送数据 read,recv
   - 服务器发送数据 send,wirte
   
   
   
   ### LT和ET模式 
   
   - LT(level triggered )水平触发，这种工作模式效率差。该模式下，当一个事件来临时，事件在未处理或处理完之前会一直被触发，监听套接字使用的都是水平触发，这样不会丢失三次握手的链接
   - ET(edge triggered)边缘触发，只对非阻塞套接字有用，当事件来临时，内核只通知一次，如果没有处理，不会再次进行通知，因此效率高。在ET模式下，为了一次把数据接收完毕，需要循环调用recv直到数据接收完毕，这需要sock_fd为非阻塞的套接字，那么当数据为空之后，recv返回-1且errno为EAGAIN
   - 内核如何实现的ET和LT? 对于ET模式，只要用户处理了该事件，内核就会把事件移除双向链表，而LT模式下，如果数据没有处理完，则不会从双向链表移除。
   - 如何选择LT和ET？  如果数据的收发有固定的格式，建议采用LT模式，这样编程简单，清晰，如果一次处理操作能够把数据处理完，那么这样模式的效率也不会低。如果没有固定格式，那么可以采用ET模式。
   
   
   
   