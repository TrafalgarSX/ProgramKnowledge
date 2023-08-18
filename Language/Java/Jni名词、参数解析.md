
**问题 3：为什么 JNI 函数名是 Java_com_xurui_HelloWorld_sayHi？** 

答：这是 JNI 函数静态注册约定的函数命名规则，当 Java 虚拟机调用 native 方法时，需要执行对应的 JNI 函数，而 JNI 函数注册讨论的就是如何确定 native 方法与 JNI 函数之间的映射关系，有两种方法：静态注册和动态注册。

静态注册采用的是基于约定的命名规则，无重载时采用「短名称」规则，有重载时采用「长名称」规则。

**问题 4：关键词 JNIEXPORT 是什么意思？** 答：JNIEXPORT 是一个宏定义，表示一个函数需要暴露给共享库外部使用时。JNIEXPORT 在 Window 和 Linux 上有不同的定义：
    
```cpp
Windows 平台 :  
#define JNIEXPORT __declspec(dllexport)
#define JNIIMPORT __declspec(dllimport)

Linux 平台：
#define JNIIMPORT
#define JNIEXPORT  __attribute__ ((visibility ("default")))

```

**问题 5：关键词 JNICALL 是什么意思？** 答：JNICALL 是一个宏定义，表示一个函数是 JNI 函数。JNICALL 在 Window 和 Linux 上有不同的定义：

```cpp
Windows 平台 :  
#define JNICALL __stdcall // __stdcall 是一种函数调用参数的约定 ,表示函数的调用参数是从右往左。

Linux 平台：
#define JNICALL

```

**问题 6：第一个参数 JNIEnv* 是什么？** 答：第一个参数是 JNIEnv 指针，**指向一个 JNI 函数表**。通过这些 JNI 函数可以让本地代码访问 Java 虚拟机的内部数据结构。

JNIEnv 指针还有一个作用，就是屏蔽了 Java 虚拟机的内部实现细节，使得本地代码库可以透明地加载到不同的 Java 虚拟机实现中去（牺牲了调用效率）。(**重要**)

**问题 7：第二个参数 jobject 是什么？** 答：第二个参数根据 native 方法是静态方法还是实例方法有所不同。对于静态 native 方法，第二个参数 jclass 代表 native 方法所在类的 Class 对象。对于实例 native 方法，第二个参数 jobject 代表调用 native 的对象

