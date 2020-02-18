## async和future

------

#### std::async

- std::async是一个函数模板，来启动一个异步任务，返回一个std::future对象

  ```c++
  # include<future>
  
  int my_thread()
  {
      int res = 0;
      //...do some thing
      return res;
  }
  
  std::future<int> result = std::async(my_thread);
  std::cout << result.get(); 
  ```

- my_thread是一个callable对象，当调用async时，**异步执行该函数**，把返回的结果存放到future对象中，因此future类模板的参数需要和my_thread函数返回类型一致。

- future对象的数据通过get()方法获取，若任务函数还没有返回则get()方法会阻塞，等待my_thread执行完毕。get()方法只能调用一次。

- future.wait()等待异步任务返回，但并不返回任何结果。

- 可以额外向std::async()传递一个参数std::launch(这是一个枚举类型)，其包含两个值std::launch::defered和std::launch::async。std::launch::defered标记若被传递到async()函数，则表异步任务不会立即执行，而是等到future对象调用wait()和get()方法，但是由于get和wait会阻塞，所以实际上async并不会创建线程，而是直接在主线程中调用my_thread。如果传入std::launch::async标记，则会立即创建一个新线程执行异步任务。默认情况下使用launch::async | launch::deferred，即由系统决定执行策略。

- 当系统资源紧张时，async默认参数情况下不会创建新线程，而是在调用get()的线程中执行异步任务

  ```c++
  std::future<int> res = std::async(std::launch::defered, my_thread);
  ```



#### std::packaged_task

- 类模板，模板参数为可调用对象的返回值和參數類型，通过std::packaged_task把各种可调用对象包装起来方便作为线程入口函数使用。

- 在线程启动之后，可以调用get_future()方法返回future对象

- packaged_task包装后的可调用对象可通过调用来调用被包装的被包装的对象，直接调用不会创建线程，而是直接在主线程执行，获取返回值同样是使用get_future()

  ```c++
  int mythread(int num)
  {
  	//...do some thing	
  }
  std::packaged_task<int(int)> my_t (mythread);
  std::thread t(std::ref(mypt), 10);
  t.join();
  std::future<int> res = mypt.get_future();
  
  ```



#### std::promise

- 类模板，作用为在某个线程中赋值，然后可以其他线程中取到结果

  ```c++
  void my_thread(std::promise<int>& temp, int num)
  {
  		//do some calc
  		num *= 10;
  		temp.set_value(num);
  		returen;
  }
  std::promise<int> my_promise;
  std::thread t(my_thread, std::ref(my_promise), 999);
  t.join();
  std::future<int> ful = my_promise.get_future();
  int res = ful.get();
  
  ```

  

