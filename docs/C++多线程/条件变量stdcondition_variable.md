## 条件变量std::condition_variable

------

###   condition_variable.wait()

```c++
std::mutex m;
std::unique_lock<mutex> guard(m);
std::condition_variable cond;
cond.wait(guard, callable);
```

- 条件变量需要和互斥量配合使用
- 如果callable返回false，那么wait()将会阻塞，**并解锁互斥量**，直到阻塞到其他线层调用notify_one() 
- 如果返回true，函数直接返回
- wait()函数也可不加第二个参数，则其效果和返回false值相同
- 当其他线程把wait()唤醒之后，那么首先wait()首先不断重新尝试获取互斥锁，如果暂时获取不到则阻塞直到获取到锁，当获取到锁(即等对互斥量加锁了)那么再次调用callable

### notify_one()和notify_all()

- notify_one()用于通知一个阻塞于wait()的线程，如果没有线程阻塞于wait()，那么将没有线程被唤醒
- notify_all()用于通知所有阻塞与wait()的线程，这样可以避免多个线程竞争时有些线程永远不会得到notify通知的情况

### 虚假唤醒

- 线程被唤醒时而条件变量尚未满足条件，这种问题常见于代码的逻辑错误，因此cond.wait(mutex, callable)的第二个参数的函数用于在唤醒之后判断条件是否满足。

