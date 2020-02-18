## shared_future和atomic

------

#### std::future_status

- 一个枚举类型，包含三个值timeout(在当前时间线程尚未返回)，ready(已返回)，deferred(表示线程尚未创建，即async()函数中传入了std::launch::deferred)

  ```c++
  std::future<int> result = async(func);
  std::future_status = result.wait(std::chrono::seconds(6));
  ```

  

#### std::shared_future

- 类模板，功能和future相同，区别在于，future的get()函数时移动语义，因此get()函数只能调用一次，而shared_future的get()进行复制，可以通过多次调用get()获取异步函数的调用结果，这样适合在多个线程中获取相同的某个值。

  ```c++
  std::future<int> result = async(func);
  //可以通过share传递future对func调用返回结果的所有权
  std::shared_future<int> res (result.shared());
  //传递之后result.valid()返回false
  ```

   

#### std::atomic原子操作

- 是一种无锁技术，在多线程中不会被打断的程序执行片段，其效率比使用互斥量高

- 原子操作只能针对一个变量(互斥锁可以针对一个代码段进行加锁)

- std::atomic是一个类模板，用来封装一个变量

- atomic原子操作对于++，--，+=， &=， |=, ^=等操作支持，其他的操作不支持，使用前需要进行一定的检验

- 定义为原子操作的变量不允许复制和赋值给其他变量，编译器内部不允许拷贝构造。如果想要获得原子操作变量中的值，使用成员方法atomic_var.load()即读操作，如要往原子操作变量则可以调用atomic_var.store(arg)

  ```c++
  //封装一个int变量，可以把该对象当做整型来使用
  std::stomic<int> variable = 0;
  for (int i = 0; i < 10000000; i++){
      variable ++;
  }
  //如果两个线程同时执行for循环的的内容，那么执行的结果时永远正确的，不会在++
  //操作执行一半时被打断的情形，且这么做的效果比使用互斥量的效率高很多
  
  ```

  