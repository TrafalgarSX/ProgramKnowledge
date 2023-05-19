### std::ref()的作用
- std::bind使用的是参数的拷贝而不是引用，当可调用对象期待入参为引用时，必须显示利用std::ref来进行引用绑定。
- 多线程std::thread的可调用对象期望入参为引用时，也必须显式通过std::ref来绑定引用进行传参

```cpp

std::mutex mtx;

// 作为线程函数，传实参时需要 std::ref(number)
void thread_function_ref(int &number) {
  mtx.lock();
  number++;
  std::cout << "Hello from thread " << number << std::endl;
  mtx.unlock();
}

void thread_function(int *number){
  mtx.lock();
  (*number)++;
  std::cout << "Hello from thread " << *number << std::endl;
  mtx.unlock();
}

int main(void) {
  int number = 0;
  std::thread thread_group[10];

	thread_function(&number);

  for (int i = 0; i < 10; i++) {
    // thread_group[i] = std::thread(thread_function, std::ref(number));
    thread_group[i] = std::thread(thread_function, &number);
  }

  for (int i = 0; i < 10; i++) {
    thread_group[i].join();
  }
  return 0;
}
```
need more google

[reference - C++ Difference between std::ref(T) and T&? - Stack Overflow](https://stackoverflow.com/questions/33240993/c-difference-between-stdreft-and-t)