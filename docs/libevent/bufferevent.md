## bufferevent

------

### bufferevent 基本概念

### 缓冲区

输入缓冲区, 输出缓冲区, evbuffer

#### 回调和水位

读取回调, 写入回调

### 水位

- 读取低水位
  - 读取操作使得输入缓冲区的数据量在此级别或者更高时，读取回调将被调用。
- 读取高水位
  - 输入缓冲区中的数据量达到此级别后，bufferevent 将停止读取，直到输入缓冲区中足够量的数据被抽取，使得数据量低于此级别。
- 写入低水位
  - 写入操作使得输出缓冲区的数据量达到或者低于此级别时，写入回调将被调用
- 写入高水位

#### 事件回调

#define BEV_EVENT_READING        0x01 /**< error encountered while reading */

#define BEV_EVENT_WRITING        0x02 /**< error encountered while writing */

#define BEV_EVENT_EOF                  0x10 /**< eof file reached */

#define BEV_EVENT_ERROR            0x20 /**< unrecoverable error encountered */

#define BEV_EVENT_TIMEOUT       0x40 /**< user-specified timeout reached */

#define BEV_EVENT_CONNECTED 0x80 /**< connect operation finished. */



## api

- bufferevent_socket_new struct bufferevent *bufferevent_socket_new(

  struct event_base *base,

  evutil_socket_t fd,

  enum bufferevent_options options);

- BEV_OPT_CLOSE_ON_FREE

  - 释放时关闭socket

- BEV_OPT_THREADSAFE

  - 线程安全，自动分配锁，可以在多线程中使用

- BEV_OPT_DEFER_CALLBACKS

  - 延迟所有回调？？防有栈溢出
    - 延迟回调不会立即调用，而是在event_loop（）调用中被排队，延迟即把事件放在队尾

- BEV_OPT_UNLOCK_CALLBACKS

  - 开启线程安全后，默认调用你传递回调函数会加锁，设置这个就不锁

### bufferevent_enable

void bufferevent_enable(struct bufferevent *bufev, short events);

void bufferevent_disable(struct bufferevent *bufev, short events);

short bufferevent*get*enabled(struct bufferevent *bufev);

EV*READ, EV*WRITE, or EV*READ|EV*WRITE

### bufferevent_setcb

- void bufferevent_setcb(struct bufferevent *bufev, bufferevent*data*cb readcb, bufferevent*data*cb writecb,

- typedef void (*bufferevent_data_cb)(struct bufferevent *bev, void *ctx);

### bufferevent_event_cb eventcb, void *cbarg);

- typedef void (*bufferevent_event_cb)(struct bufferevent *bev,  short events, void *ctx);

### bufferevent_set_timeouts

void bufferevent_set_timeouts(struct bufferevent *bufev, const struct timeval _timeout_read, const struct timeval _timeout_write);

### bufferevent_setwatermark

void bufferevent_setwatermark(struct bufferevent *bufev, short events, size_t lowmark, size_t highmark);

### bufferevent_get_input

struct evbuffer *bufferevent_get_input(struct bufferevent *bufev);

### bufferevent_get_output

evbuffer *bufferevent_get_output(struct bufferevent *bufev);

### bufferevent_write

int bufferevent_write(struct bufferevent *bufev, const void *data, size_t size);

### bufferevent_read

size*t bufferevent_read(struct bufferevent *bufev, void *data, size_t size);

int bufferevent_read_buffer(struct bufferevent *bufev, struct evbuffer *buf);

### void bufferevent_free(struct bufferevent *bev);

- 这个函数释放一个缓冲事件。bufferevent是内部引用计数的，因此如果bufferevent已经挂起并被延迟

  释放回调时，在回调完成之前不会删除它。

  但是，bufferevent_free()函数会尝试尽快释放bufferevent。如果有等待写入的数据

  bufferevent，它可能不会在释放bufferevent之前被刷新。

  如果设置了BEV_OPT_CLOSE_ON_FREE标志，并且此bufferevent具有与其关联的套接字或底层bufferevent

  当释放bufferevent时，该传输将关闭。