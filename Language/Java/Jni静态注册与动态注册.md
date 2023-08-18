[NDK 系列（6）：说一下注册 JNI 函数的方式和时机 - 掘金](https://juejin.cn/post/7125021894562349092)

![[静态注册与动态注册.png]]

### 静态注册
---
#### 静态注册使用方法
静态注册采用的是**基于「约定」的命名规则**，通过 `javah` 可以自动生成 native 方法对应的函数声明（IDE 会智能生成，不需要手动执行命令）。例如：

```cpp
package com.xurui.hellojni;

public class HelloWorld {
    public native void sayHi();
}
```

执行命令：`javac -h . HelloWorld.java`（将 javac 和 javah 合并），对应的 JNI 函数：

```cpp
...
JNIEXPORT void JNICALL Java_com_xurui_hellojni_HelloWorld_sayHi
(JNIEnv *, jobject);
...
```

静态注册的命名规则分为「无重载」和「有重载」2 种情况：无重载时采用「短名称」规则，有重载时采用「长名称」规则。

- **短名称规则（short name）：** `Java_[类的全限定名 (带下划线)]_[方法名]` ，其中类的全限定名中的 `.` 改为 `_` ；
- **长名称规则（long name）：** 在短名称的基础上后**追加两个下划线（`__`）和参数描述符**，以区分函数重载。

这里解释下为什么有重载的时候要拼接参数描述符的方式来呢？因为 C 语言是没有函数重载的，无法根据参数来区分函数重载，所以才需要拼接后缀来消除重载。

#### 静态注册原理分析
现在，我们来分析下静态注册匹配 JNI 函数的执行过程。由于没有找到直接相关的资料和函数调用入口，我是以 `loadLibrary()` 加载 so 库的执行流程为线索进行分析的，最终定位到 `FindNativeMethod()` 这个方法，从内容看应该没错。

java_vm_ext.cc
```cpp
// 共享库列表
std::unique_ptr<Libraries> libraries_;

// 搜索 native 对应 JNI 函数的函数指针
void* FindNativeMethod(Thread* self, ArtMethod* m, std::string& detail) {

    // 1、获取 native 方法对应的短名称与长名称
    std::string jni_short_name(m->JniShortName());
    std::string jni_long_name(m->JniLongName());

    // 2、在已经加载的 so 库中搜索
    void* native_code = FindNativeMethodInternal(self,
                                                 declaring_class_loader_allocator,
                                                 shorty,
                                                 jni_short_name,
                                                 jni_long_name);
    return native_code;
}

// 2、在已经加载的 so 库中搜索
void* FindNativeMethodInternal(Thread* self,
                               void* declaring_class_loader_allocator,
                               const char* shorty,
                               const std::string& jni_short_name,
                               const std::string& jni_long_name) {
    for (const auto& lib : libraries_) {
        SharedLibrary* const library = lib.second;

        // 2.1 检查是否为相同 ClassLoader
        if (library->GetClassLoaderAllocator() != declaring_class_loader_allocator) {
            continue;
        }

        // 2.2 先搜索短名称
        const char* arg_shorty = library->NeedsNativeBridge() ? shorty : nullptr;
        void* fn = dlsym(library, jni_short_name)

        // 2.3 再搜索长名称
        if (fn == nullptr) {
            fn = dlsym(library, jni_long_name)
        }
        if (fn != nullptr) {
            return fn;
        }
    }
    return nullptr;
}
```

art_method.cc
```cpp
// 1、获取 native 方法对应的短名称与长名称

// 短名称
std::string ArtMethod::JniShortName() {
    return GetJniShortName(GetDeclaringClassDescriptor(), GetName());
}

// 长名称
std::string ArtMethod::JniLongName() {
    std::string long_name;
    long_name += JniShortName();
    long_name += "__";

    std::string signature(GetSignature().ToString());
    signature.erase(0, 1);
    signature.erase(signature.begin() + signature.find(')'), signature.end());

    long_name += MangleForJni(signature);

    return long_name;
}
```

descriptors_names.cc
```cpp
std::string GetJniShortName(const std::string& class_descriptor, const std::string& method) {
    // 此处为短名称的计算逻辑
}
```

上面的代码已经非常简化了，主要流程如下：

- 1、计算 native 方法的短名称和长名称；
- 2、确定定义 native 方法类的类加载器，在已经加载的 so 库 `libraries_` 中搜索 JNI 函数。如果事先没有加载 so 库，则自然无法搜索到，将抛出 `UnsatisfiedLinkError` 异常；
- 3、建立内部数据结构，建立 Java native 方法与 JNI 函数的函数指针的映射关系；
- 4、后续调用 native 方法，则直接调用已记录的函数指针。

关于加载 so 库的流程在 [4、so 文件加载过程分析](https://juejin.cn/post/6892793299427491854 "https://juejin.cn/post/6892793299427491854") 这篇文章里讲过，加载后的共享库就是存储在 `libraries_` 表中。

![[静态注册原理.png]]

### 动态注册
---
静态注册是在**首次调用** Java native 方法时搜索对应的 JNI 函数，而动态注册则是**提前手动建立映射关系**，并且不需要遵守静态注册的 JNI 函数命名规则。

#### 动态注册使用方法
动态注册需要使用 `RegisterNatives(...)` 函数，其定义在 `jni.h` 文件中：
jni.h
```cpp
struct JNINativeInterface {
		// 注册
		// 参数二：Java Class 对象的表示
		// 参数三：JNINativeMethod 结构体数组
		// 参数四：JNINativeMethod 结构体数组长度
		jint        (*RegisterNatives)(JNIEnv*, jclass, const JNINativeMethod*, jint);
		// 注销
		// 参数二：Java Class 对象的表示
    jint        (*UnregisterNatives)(JNIEnv*, jclass);
};

typedef struct {
    const char* name;      // Java 方法名
    const char* signature; // Java 方法描述符
    void*       fnPtr;     // JNI 函数指针
} JNINativeMethod;

```

e.g.
```cpp
// 需要注册的 Java 层类名
#define JNIREG_CLASS "com/xurui/MainActivity"

// JNINativeMethod 结构体数组
static JNINativeMethod gmethod[] = {
        {"onStart","()I",(void*)onStart}, // JNINativeMethod 结构体
};

// 加载 so 库的回调
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM* vm, void* reserved)
{
    JNIEnv* env = NULL;
    jint result = -1;

    if (vm->GetEnv((void**) &env, JNI_VERSION_1_6) != JNI_OK) {
        return -1;
    }
    assert(env != NULL);

		// 执行动态注册
    if (!registerNatives(env)) {
        return -1;
    }

    result = JNI_VERSION_1_6;

    return result;
}

// 动态注册
static int registerNatives(JNIEnv* env)
{
		// 调用工具方法完成注册
    if (!registerNativeMethods(env,JNIREG_CLASS, gmethod, sizeof(gmethod) / sizeof(gmethod[0])))
        return JNI_FALSE;

		// JNINativeMethod 结构体数组
    JNINativeMethod methods[] = {
            {"onStop","()V",(void*)onStop}, // JNINativeMethod 结构体
    };
		// 调用工具方法完成注册
    if (!registerNativeMethods(env,JNIREG_CLASS, smethod,sizeof(smethod) / sizeof(smethod[0])))
        return JNI_FALSE;
    return JNI_TRUE;
}

// 一个工具方法
static int registerNativeMethods(JNIEnv* env, const char* className, JNINativeMethod* gmethod, int numMethods)
{
		// 根据类名获取 jclass 对象
    jclass clazz = env->FindClass(className);
    if (clazz == NULL) {
        return JNI_FALSE;
    }
		// 调用 RegisterNatives()
    if (env->RegisterNatives(clazz, gmethod, numMethods) < 0) {
        return JNI_FALSE;
    }

    return JNI_TRUE;
}
```

代码不复杂，简单解释一下：

- 1、`registerNativeMethods` 只是一个工具函数，简化了 FindClass 这一步；
- 2、`methods` 是一个 JNINativeMethod 结构体数组，定义了 Java native 方法与 JNI 函数的映射关系，需要记录 Java 方法名、Java 方法描述符、JNI 函数指针；
- 3、`RegisterNatives` 函数是最终的注册函数，需要传递 jclass、JNINativeMethod 结构体数组和数组长度。

#### 动态注册原理分析
**RegisterNatives 方式的本质是直接通过结构体指定映射关系**，而不是等到调用 native 方法时搜索 JNI 函数指针，因此**动态注册的 native 方法调用效率更高**。此外，还能减少生成 so 库文件中导出符号的数量，则能够优化 so 库文件的体积。更多信息见 [Android 对 so 体积优化的探索与实践](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2F7YVuouHAq2OfrowhoHVmnQ "https://mp.weixin.qq.com/s/7YVuouHAq2OfrowhoHVmnQ") 中 “精简动态符号表” 章节。

### 注册 JNI 函数的时机
---
总结一下注册 JNI 函数的时机，主要分为 3 种：

|注册时机|注册方式|描述|
|---|---|---|
|1、在第一次调用该 native 方法时|静态注册|虚拟机会在 JNI 函数库中搜索函数指针并记录下来，后续调用不需要重复搜索|
|2、加载 so 库时|动态注册|加载 so 库时会自动回调 JNI_OnLoad 函数，在其中调用 RegisterNatives 注册|
|3、提前注册|动态注册|在加载 so 库后，调用该 native 方法前，通过静态注册的 native 函数触发 RegisterNatives 注册。例如在 App 启动时，很多系统源码会提前做一次注册|

以 Android 虚拟机源码为例：在 App 进程启动流程中，在**创建虚拟机后会执行一次 JNI 函数注册**。我们在很多 Framework 源码中可以看到 native 方法，但找不到调用 `System.loadLibrary(...)` 的地方，其实是因为在虚拟机启动时就已经注册完成了。

AndroidRuntime.cpp
```cpp
void AndroidRuntime::start(const char* className, const Vector<String8>& options, bool zygote) {
    ...
    if (startReg(env) < 0) {
        ALOGE("Unable to register all android natives\\n");
    }
    ...
}

// start -> startReg：
int AndroidRuntime::startReg(JNIEnv* env) {
    androidSetCreateThreadFunc((android_create_thread_fn) javaCreateThreadEtc);
    env->PushLocalFrame(200);
		// 执行 JNI 注册
    if (register_jni_procs(gRegJNI, NELEM(gRegJNI), env) < 0) {
        env->PopLocalFrame(NULL);
        return -1;
    }
    env->PopLocalFrame(NULL);
    return 0;
}

// startReg->register_jni_procs：
static int register_jni_procs(const RegJNIRec array[], size_t count, JNIEnv* env) {
		// 遍历二维数组
    for (size_t i = 0; i < count; i++) {
        // 执行 JNI 注册
        if (array[i].mProc(env) < 0) {
            return -1;
        }
    }
    return 0;
}

// JNINativeMethod 结构体数组的二维数组
static const RegJNIRec gRegJNI[] = {
    REG_JNI(register_com_android_internal_os_RuntimeInit),
    REG_JNI(register_com_android_internal_os_ZygoteInit_nativeZygoteInit),
    REG_JNI(register_android_os_SystemClock),
    REG_JNI(register_android_util_EventLog),
    ...
}

struct RegJNIRec {
    int (*mProc)(JNIEnv*);
}

// 返回值为 JNINativeMethod 结构体数组
int register_com_android_internal_os_RuntimeInit(JNIEnv* env)
{
    const JNINativeMethod methods[] = {
        { "nativeFinishInit", "()V",
            (void*) com_android_internal_os_RuntimeInit_nativeFinishInit },
        { "nativeSetExitWithoutCleanup", "(Z)V",
            (void*) com_android_internal_os_RuntimeInit_nativeSetExitWithoutCleanup },
    };
    return jniRegisterNativeMethods(env, "com/android/internal/os/RuntimeInit",
        methods, NELEM(methods));
}
```


### 总结
---
总结一下静态注册和动态注册的区别：

- 1、静态注册基于命名约定建立映射关系，而动态注册通过 `JNINativeMethod` 结构体建立映射关系；
- 2、静态注册在首次调用该 native 方法搜索并建立映射关系，而动态注册会在调用该 native 方法前建立映射关系；
- 3、静态注册需要将所有 JNI 函数暴露到动态符号表，而动态注册不需要暴露到动态符号表，可以精简 so 文件体积。