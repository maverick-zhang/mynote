## mutex互斥量

------

### std::mutex

1. 包含在头文件\<mutex\>中，包含两个成员方法lock()和unlock()，必须成对使用，若已被锁住，则再次调用lock()会阻塞到被解锁
2. mutex在windows系统对于的概念为临界区，所不同的是，临界区创建之后需要初始化，进入临界区相当于锁定互斥量，离开临界区则对应为互斥量的解锁。在同一个线程中，多次进入(中间没有离开)相同的临界区是允许的，且需要离开相同次数。互斥量只能一次加锁，一次解锁配对使用。

### recursive_mutex

1. 解决mutex在同一线程同一互斥量不能重复加锁的问题，功能类似windows临界区，递归次数有限制，不能无限递归
2. 但是尽量使用mutex，考虑代码是否右优化的空间避免多次加锁

### timed_mutex和recursive_timed_mutex

1. 带超时的互斥量。对象方法try_lock_for(time_span)，在一段时间内尝试获取锁，若成功拿到锁返回true，超过时间后表示未拿到返回false；try_lock_until(time_point)，的参数为未来的时间点，表示在这个时间点之前尝试加锁。需要配合std::chrono使用。

### std::lock_guard\<std::mutex\>

1. 是一个智能锁对象，构造时接受一个互斥量对象std::mutex(或recursive_mutex)，在对象构造时自动调用mutex.lock()进行加锁，在对象析构时调用mutex.unlock()

2. 构造时可以接收第二个参数std::adopt_lock，这是一个结构对象，相当于一个flag告诉lock_guard在构造对象时不调用mutex.lock()

   ```c++
   #include <mutex>
   std::mutex m;
   std::lock_guard<std::mutex> my_guard(m, std::adopt_lock);
   ```



### std::unique_lock\<std::mutex\>

1. 智能锁对象，相对于lock_guard提供更细的控制。在只有一个参数(mutex对象)时，其和lock_guard的行为一致，即在构造时调用mutex.lock()，以及在析构时调用mutex.unlock()
2. 第二个参数可以为adopt_lock，以为构造对象时不调用lock()
3. 若第二个参数为std::try_to_lock()则在调用mutex.lock()不会阻塞，可以通过成员方法owns_lock()判断是否lock成功
4. 若第二个参数为std::defer_lock，则表示传入的mutex没有lock过，需要自己在后面的合适位置手动调用lock()方法
5. unique_lock包含lock()，unlock()成员方法，其实就是mutex的上述方法的封装
6. 成员方法release()，放弃对mutex的所有权，功能和unique_ptr类似。当需要进行所有权转移时，需要使用转移语义(std::move方法或通过返回临时对象)