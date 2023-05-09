### What is the Python Global Interpreter Lock?
---
The Python Global Interpreter Lock or [GIL](https://wiki.python.org/moin/GlobalInterpreterLock), in simple words, is a mutex (or a lock) that allows only one [thread](https://realpython.com/intro-to-python-threading/) to hold the control of the Python interpreter.

This means that only one thread can be in a state of execution at any point in time. The impact of the GIL isn’t visible to developers who execute single-threaded programs, but **it can be a performance bottleneck in CPU-bound and multi-threaded code.**

Since the GIL allows only one thread to execute at a time even in a multi-threaded architecture with more than one CPU core, **the GIL has gained a reputation as an “infamous” feature of Python.**

### GIL锁对python多线程的影响
---
💡Python doesn't support multithreading because threading packages couldn't let us use the multiple CPU cores.

CPython实现细节：在 CPython 中，由于存在 [全局解释器锁](https://docs.python.org/zh-cn/3/glossary.html#term-global-interpreter-lock)，**同一时刻只有一个线程可以执行 Python 代码（虽然某些性能导向的库可能会去除此限制）**。 如果你想让你的应用更好地利用多核心计算机的计算资源，推荐你使用 [`multiprocessing`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#module-multiprocessing "multiprocessing: Process-based parallelism.") 或 [`concurrent.futures.ProcessPoolExecutor`](https://docs.python.org/zh-cn/3/library/concurrent.futures.html#concurrent.futures.ProcessPoolExecutor "concurrent.futures.ProcessPoolExecutor")。 但是，**如果你想要同时运行多个 I/O 密集型任务，则多线程仍然是一个合适的模型。**

If your program is **IO-bound**, both multithreading and multiprocessing in Python will work smoothly. However, If the code is **CPU-bound** and your machine has multiple cores, multiprocessing would be a better choice.

Python multithreading works great for creating a responsive graphic user interface and for handling short web requests where one thread handles the GUI actions and the other processes the files one at a time.

TODO 补充阅读:
[What Is the Python Global Interpreter Lock (GIL)? – Real Python](https://realpython.com/python-gil/)

### Why Python Developers Uses GIL?
---
Python provides the unique reference counter feature, which is used for memory management. The reference counter counts the total number of references made internally in Python to assign a value to a data object. When the reference counts reach zero, the assigned memory of the object is released. 

**The main concern with the reference count variable is that it can be affected when two or three threads trying to increase or decrease its value simultaneously.** It is known as race condition. If this condition occurs, it can be caused leaked memory that is never released. It may crash or bugs in the Python program.