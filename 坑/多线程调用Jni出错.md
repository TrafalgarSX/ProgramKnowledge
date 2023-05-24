### 问题与现象
---
工作中要模拟使用Jni调用动态库，多线程调用一个JNI接口，使其可以让动态库多线程发送签名的网络请求，测试服务端是否可以处理多线程签名的请求，是否会出现崩溃问题。

IOS之前出现过多线程调用动态库接口（不通过jni)，导致服务端崩溃的问题。与服务端一起完成了这个问题的解决方案，想在Android平台上测试的时候，当通过jni多线程调用动态库接口的时候Android APP一直出现崩溃问题，无法复现IOS的情境。

出现了这样的调用栈：
```
SIGABRT信号
abort()
JniAboitV
ScopeCheck::CheckThread 
_JNIEnv::FindClass 
```

chatgpt对出现此调用栈的回答：
这个调用栈显示了程序在执行期间发生了崩溃。它的具体原因可能与线程相关，因为它调用了ScopeCheck::CheckThread函数。这个函数可能是检查线程的函数，如果线程不正确，它可能会导致程序崩溃。

因此，可能需要检查线程相关的代码，以确保线程的正确性。还需要检查JNI调用的代码，以**确保它们正确地使用了JNIEnv对象和类引用。**

### 原因与解决方法
---
原因：
Android下枚举设备的时候会回调JAVA层去枚举sim卡设备（IOS没有这个权限，不需要），当native回调JAVA时， 需要使用上下文 JNIEnv。

多线程调用JNI可能会导致_JNIEnv错误，**因为JNIEnv对象是线程特定的。每个线程都应该有自己的JNIEnv对象。** 如果多个线程共享同一个JNIEnv对象，可能会导致_JNIEnv错误，从而导致程序崩溃或其他问题。

动态库中每次调用接口都会将JNIEnv都保存在一个全局变量中，最后回调java的时候才会将这个全局变量强转为JNIEnv对象，并使用这个对象进行回调，这就会导致多线程调用jni接口的时候，后一个线程的JNIEnv对象会把前一个线程的JNIEnv对象覆盖，前一个线程回调java时就会使用错误的JNIEnv对象，导致出现崩溃问题。

解决方法：
为了避免_JNIEnv错误，**可以使用JavaVM来获取当前线程的JNIEnv对象**。JavaVM是跨线程的，因此可以安全地将JNIEnv对象传递给不同的线程。在JNI中，可以使用JavaVM::AttachCurrentThread函数来获取当前线程的JNIEnv对象，并使用JavaVM::DetachCurrentThread函数来释放JNIEnv对象。

示例代码：
```cpp
JavaVM* jvm; // 全局Java虚拟机对象

JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM* vm, void* reserved) {
    jvm = vm; // 保存Java虚拟机对象
    return JNI_VERSION_1_6;
}

void doSomething() {
    JNIEnv* env;
	// cpp 可以这样调用
    jint res = jvm->AttachCurrentThread((void**)&env, NULL);
	// c中应该这样调用， 否则会有提示JNIInvokeInterface is not a structure or union
	jint res = (*jvm)->AttachCurrentThread(jvm, (void**)&env, NULL);

    if (res != JNI_OK) {
        // 处理AttachCurrentThread失败的情况
        return;
    }

    // 在这里使用JNIEnv对象执行JNI操作
    jvm->DetachCurrentThread();
}
```

在上面的示例代码中，我们首先在JNI_OnLoad函数中保存了Java虚拟机对象。在doSomething函数中，我们首先调用AttachCurrentThread函数获取当前线程的JNIEnv对象，并将其保存在env变量中。在使用JNIEnv对象执行JNI操作后，我们最后调用DetachCurrentThread函数释放JNIEnv对象。 

需要注意的是，AttachCurrentThread函数的第一个参数是一个指向JNIEnv指针的指针。这是因为AttachCurrentThread函数需要在其中存储JNIEnv对象的地址。如果AttachCurrentThread函数成功，它将修改该指针以指向当前线程的JNIEnv对象。

💡`JNI_OnLoad`是一个JNI库函数，**它在JNI库被加载时自动调用**。当Java应用程序加载使用JNI库时，JNI库会被加载到Java虚拟机中。在JNI库被加载时，Java虚拟机会自动调用JNI_OnLoad函数，并将Java虚拟机对象的指针作为参数传递给该函数。因此，**JNI_OnLoad函数不是在Java层调用的，而是在JNI库层自动调用的。**

在JNI_OnLoad函数中，你可以执行一些初始化操作，例如保存Java虚拟机对象的指针，注册JNI函数等等。这些操作对于JNI库的正确使用非常重要。另外，你还可以在JNI_OnUnload函数中执行清理操作。`JNI_OnUnload`函数在JNI库被卸载时自动调用，你可以在该函数中释放JNI库中使用的资源。