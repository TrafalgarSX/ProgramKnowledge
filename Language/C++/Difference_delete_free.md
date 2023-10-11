### delete and free() in C++

**delete** and **free()** in C++ have similar functionalities but they are different.
功能类似， 但实际上有区别。

- delete is operator
- free() is a function

In C++, the **delete** operator should only be used for deallocating the memory allocated either using the **new** operator or for a NULL pointer

**free()** should only be used for deallocating the memory allocated either using **malloc(), calloc(), realloc() or for a NULL pointer.**

表格对比：

|                                                          delete                                                           | free()                                                                                                                         |
|:-------------------------------------------------------------------------------------------------------------------------:| ------------------------------------------------------------------------------------------------------------------------------ |
|                                                    It is an operator.                                                     | It is a library function.                                                                                                      |
|                                          It de-allocates the memory dynamically.                                          | It destroys the memory at runtime.                                                                                             |
|   It should only be used for deallocating the memory allocated either using the **new** operator or for a NULL pointer.   | It should only be used for deallocating the memory allocated either using malloc(), calloc(), realloc() or for a NULL pointer. |
|                        This operator calls the destructor before it destroys the allocated memory.                        | This function only frees the memory from the heap. It does not call the destructor.                                            |
| It is comparatively solwer because it invokes the destructor for the object being deleted before deallocating the memory. | It is faster than delete operator.                                                                                             |


❗The most important reason why free() should not be used for de-allocating memory allocated using **new** is that, **it does not call the destructor of that object while delete operator calls the destructor to ensure cleanup and resource deallocation for objects.**


delete会调用析构函数， free不会，所以非POD类型，应该用 new 分配内存， delete 回收内存。
