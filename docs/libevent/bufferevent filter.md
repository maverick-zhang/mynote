## bufferevent filter

### 缓冲数据存储 evbuffer

- #### evbuffer_remove

  int evbuffer_remove(struct evbuffer \*buf, void \*data, size*t datlen);

- #### evbuffer_add

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