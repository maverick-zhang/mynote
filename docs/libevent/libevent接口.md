## libevent接口和配置

1. 环境配置和初始化 event_base_new
2. evutil_soket函数（对socket api的封装）
   - evutil_make_socket_nonblocking
   - evutil_make_listen_socket_reusable  设置SO_REUSEADDR
   - evutil_close_socket
3. 事件io处理 event_new  需要传递socket文件描述符
4. 缓存io bufferevent
5. 循环 event_base_dispatch

### 创建配置

1. event_config * conf = event_config_new()；
2. event_base *base = event_base_new_with_config(conf);
3. event_config_free(conf);  //要释放内存

- event_config_set_flag(struct event_conf * conf, int event_baseflag);
  - event_base_config_flag
    - EVENT_BASEFLAG_NOLOCK
      - 不要为event_base 分配锁。设置这个选项可以为
      - event*base 节省一点用于锁定和解锁的时间，但是让在多个线程中访问event*base
      - 成为不安全的。
    - EVENT_BASEFLAG_IGNORE_ENV
      - 选择使用的后端时，不要检测EVENT_*环境
      - 变量。使用这个标志需要三思：这会让用户更难调试你的程序与libevent 的交互。
    - EVENT_BASEFLAG_STARTUP_IOCP
      - 仅用于Windows,启用任何必需的IOCP 分发逻辑
        - iocp
          - event_config_set_num_cpus_hint
          - event_config_set_flag(cfg, EVENT_BASE_FLAG_STARTUP_IOCP)
          - evthread_use_windows_threads();
    - EVENT_BASEFLAG_NOCACHE_TIME
      - 不是在事件循环每次准备执行超时回调时
      - 检测当前时间，而是在每次超时回调后进行检测。注意：这会消耗更多的CPU 时间。
    - EVENT_BASEFLAG_EPOLL_USE_CHANGELIST
      - epoll下有效，防止同一个fd多次激发事件，fd如果做复制会有bug
    - EVENT_BASEFLAG_PRECISE_TIMER
      - 默认使用系统最快的记时机制，如果系统有较慢且更精确的则采用
- event_config_avoid_method(struct event_config *cfg, const char *method);
  - 可以选择不使用方法，比如在linux下不使用"epoll"和"poll"从而选择"select"
  - 在默认情况下，linux方法的选择优先权为epoll>poll>select
- event_config_require_features
  - event_method_feature
    - EV_FEATURE_ET = 0x01,
      - 边沿触发的后端
    - EV_FEATURE_O1 = 0x02,
      - 要求添加、删除单个事件，或者确定哪个事件激活的操作是O（1）复杂度的后端
    - EV_FEATURE_FDS = 0x04,
      - 要求支持任意文件描述符，而不仅仅是套接字的后端
    - EV_FEATURE_EARLY_CLOSE = 0x08
      - 检测连接关闭事件。您可以使用它来检测连接何时关闭，而不必从连接中读取所有挂起的数据。并非所有后端都支持EV_CLOSED。
      - 允许您使用EV_CLOSED检测连接关闭，而不需要读取所有挂起的数据。
      - 无法在所有内核版本上