## bufferevent filter

### 缓冲数据存储 evbuffer

- #### evbuffer_remove 从buffer拿出原始数据 buf = src

  int evbuffer_remove(struct evbuffer \*buf, void \*data, size*t datlen);

- #### evbuffer_add  把过滤后的数据放到buffer buf = dst

  int evbuffer_add(struct evbuffer \*buf, const void \*data, size*t datlen);

### 创建过滤器bufferevent_filter_new

#### struct bufferevent *bufferevent_filter_new(

1. #### struct bufferevent *underlying,

2. #### bufferevent_filter_cb input_filter,

   - 返回值：typedef enum bufferevent_filter_result (*bufferevent_filter_cb)(

     - enum bufferevent_filter_result {

       BEV_OK = 0,    //写入了任何数据

       BEV_NEED_MORE = 1,   //没有写入

       BEV_ERROR = 2	};  //错误

   - struct evbuffer *source, struct evbuffer *destination, **

   -    **ev*ssize*t dst_limit,   高水位或者速率限制

   - enum bufferevent_flush_mode mode,
   
     - BEV_NORMAL  在方便转换的基础上写入尽可能多的数据
   
     
     - BEV_FLUSH  表示写入尽可能多的数据
     
     - BEV_FINISHED  在流的末尾执行额外的清理操作
     
   - void *ctx   
   
3. #### bufferevent_filter_cb output_filter,


4. #### int options,

   - ​	BEV_OPT_CLOSE_ON_FREE

5. #### void (*free_context)(void *),

   - 清理函数	

6. #### void *ctx



### bufferevent filter zlib压缩通信

#### evbuffer_peek

int evbuffer_peek(struct evbuffer \*buffer, ev_ssize_t len, struct evbuffer_ptr \*start_at,

struct evbuffer_iovec \*vec*out, int n_vec);   //只去查看buffer中的数据而不处理，场景：压缩后的数据尚未收取完不足以解压缩

- struct evbuffer_iovec {
- void *iov_base;
- size*t iov*len;
- };

#### evbuffer_reserve_space

int evbuffer_reserve_space(struct evbuffer *buf, ev_ssize_t size,

- 扩展缓冲区以至少提供size 字节的空间

### struct evbuffer_iovec *vec, int n*vecs);

- Must be at least 1; 2 is more efficient

写入到向量中的数据不会是缓冲区的一部分，直到调用evbuffer_commit*space()

- evbuffer_commit*space(buf, evbuffer_iovec *vec, 1);

#### int evbuffer_drain(struct evbuffer \*buf, size*t len);

清空数据，直接读取缓冲数据时用到，减少复制

### Zlib压缩

- deflateInit（z_stream*,int level）
  - Z*DEFAULT*COMPRESSION
  - Z*BEST*COMPRESSION
  - Z*BEST*SPEED
  - returns Z_OK
- int deflate (z_stream* strm, int flush)
  - returns Z_OK
  - Z_SYNC_FLUSH
    - 所有挂起的输出都是刷新到输出缓冲区，并且输出在字节边界上对齐

### 解压缩

- int inflate (z_stream* strm, int flush)
  - Z*SYNC*FLUSH将输出尽可能地输出到输出缓冲区
- int inflateInit（z_stream *strm）

### z_stream

- uInt avail_in
  - 输入空间大小
    - 处理后就是剩余未处理的大小
- Bytef *next_in;
  - 输入空间地址
- uInt avail_out
  - 输出空间大小
    - 处理后就是剩余输出空间大小
- Bytef *next_out;
  - 输出空间地址