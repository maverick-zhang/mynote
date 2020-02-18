## event loop

------

## 运行循环

### int event_base_loop(struct event_base *base, int flags);

- \#define EVLOOP_ONCE 0x01
  - 等待一个事件运行，直到没有活动的事件就退出
- \#define EVLOOP_NONBLOCK 0x02
  - 有活动事件处理，没有活动事件，立刻返回
- \#define EVLOOP_NO_EXIT_ON_EMPTY 0x04
  - 没有添加事件也不返回



## 停止循环

### int event_base_loopexit(struct event_base *base,const struct timeval *tv);

- 运行完所有激活事件的回调之才退出
- 事件循环没有运行时，下一轮回调完成后立即停止

### int event_base_loopbreak(struct event_base *base);

- 执行完当前正在处理的事件后立即退出，如无操作则立即退出