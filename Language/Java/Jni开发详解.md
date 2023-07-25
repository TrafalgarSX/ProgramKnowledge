### JNI(Java Native Interface)
---
JNI 全称为 Java Native Interface，是一种允许 Java 代码与本地（Native）代码交互的技术。JNI 提供了一组 API，可以使 Java 程序访问和调用本地方法和资源，也可以使本地代码访问和调用 Java 对象和方法。 此方案需要使用 JNI 进行双向调用。

#### JavaC
步骤：
    设计规划功能、接口
    Java 声明 Native 方法
    按照 JNI 标准实现方法，并通过 System.loadLibrary()加载

```java
public class TestJNI {
   static {
      System.loadLibrary("xxx.so"); // 加载动态链接库
   }

   // 声明本地方法
   private native void PrintHelloWorld();

   // 静态方法
   public static native String GetVersion();

}

// C实现函数
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM *vm, void *reserved) { ... } // so初始化回调函数
JNIEXPORT void JNICALL JNI_OnUnload(JavaVM *jvm, void *reserved) { ... } // so卸载回调函数

// 实现
包名_PrintHelloWorld(JNIEnv *env, jobject thiz) { ... }
包名_GetVersion(JNIEnv *env, jclass clazz) { ... }

```

#### 使用场景和优势

- java虽然跨平台，但仍然运行在具体平台(windows，linux)之上，对于需要操作硬件的功能，必须**通过系统的C/C++方法对硬件进行直接操作**，比如打开文件，java层必须调用系统的open方法（linux是open，windows是openFile）才能打开文件，这个时候就涉及到java代码如何调用C/C++代码的问题
    
- 在一些拥有复杂算法的场景 **（音视频编解码，图像绘制等）**，java的执行效率远低于C/C++的执行效率，使用jni技术，在java层调用C/C++代码，可以提高程序的执行效率，最大化利用机器的硬件资源。
    
- native层的代码往往更加安全，**反编译so文件比反编译jar文件要难得多**，所以，我们往往把涉及到**密码密钥相关**的功能用C/C++实现，然后java层通过jni调用

java运行在jvm，jvm本身就是使用C/C++编写的，因此jni只需要在java代码、jvm、C/C++代码之间做切换即可

![[Jni调用详解.png]]

### 使用步骤
---
1. 用idea创建一个java工程，并创建JNIDemo.java文件
2. 在`JNIDemo.java`文件中声明native方法`helloJni()`
```java
public class JNIDemo {
    public static native String helloJni();
}
```
3. 使用`javac`命令编译`JNIDemo.java`，生成`JNIDemo.class`文件
4. 使用`javah`命令生成`JNIDemo.h`文件
5. 使用clion创建C++ library项目，并复制刚刚生成的`com_jason_jni_JNIDemo.h`头文件到项目根目录。 注意：要创建动态库，Java才能调用
💡 3-5步不是必要的， 可以创建了native方法的类后，可以自己直接编写相关的JNI函数（函数名自己写就行，没必要专门生成） 
6. 创建`JNIDemo.cpp`文件，实现`helloJni()`方法（需要导入jni.h头文件， 在jdk当中）
7. 编译代码，生成动态库文件
8. 在刚刚的java项目的根目录中创建`libs`文件夹，并将其设置为资源文件夹，然后将生成的`libjnidemo.so`文件拷贝到该目录中。注意`libs`目录的图标一定要是资源文件夹的样式，不是普通文件夹的样式，然后将`libjnidemo.so`文件拷贝到该目录下
9. 在java代码中通过`System.loadLibrary()`加载so文件
```java
public class JNIDemo {
    static {
        System.loadLibrary("jnidemo");
    }
    public static native String helloJni();

    public static void main(String[] args) {
        System.out.println(helloJni());
    }
}
```

上面通过一个简单的案例讲解了jni的使用流程，从中不难看出，大部分步骤都是固定的，唯一不固定的是JNIDemo.cpp的内容，这个取决于实际的需求。而**在新版的Android Studio当中已经把这些固定流程封装成了模板操作**，我们可以一键生成头文件和源文件，开发者只需要关注源文件的功能实现即可。

![[Android_JNI.png]]

### JNI API详解
---
java和C/C++通信是通过jni来完成的，那么在jni方法中就涉及到**对java变量的访问**（变量类型包括基本数据类型和引用数据类型），对**java方法的调用**，**java对象的创建**等，而java语法跟jni语法不一定是一 一对应的，比如，java中叫`boolean`，jni中叫`jboolean`，那怎么解决这个问题呢，

#### 数据类型
jni给我们提供了若干个映射表，将java中的类型与jni中的类型进行了一 一映射，其中包括基本数据类型映射，引用数据类型映射，方法签名(包含参数和返回值)映射，以下是这三个映射表：
基本数据类型映射表
![[基本数据类型.png]]

引用数据类型映射表
![[引用数据类型.png]]

方法签名
![[方法签名.png]]

#### 方法调用
接下来就可以在jni方法中访问java成员了，同样的，jni给我们提供了一系列访问java成员的API，具体如下：

###### Jni访问调用对象

| 方法名         | 作用                                 |
| -------------- | ------------------------------------ |
| GetObjectClass | 获取调用对象的类，我们称其为target   |
| FindClass      | 根据类名获取某个类，我们称其为target |
| IsInstanceOf   | 判断一个类是否为某个类型             |
| IsSameObject   | 是否指向同一个对象                   |

###### Jni访问Java成员变量的值

| 方法名      | 作用                                                       |
| ----------- | ---------------------------------------------------------- |
| GetFieldId  | 根据变量名获取target中成员变量的ID                         |
| GetIntField | 根据变量ID获取int变量的值，对应的还有byte，boolean，long等 |
| SetIntField | 修改int变量的值，对应的还有byte，boolean，long等           |

###### Jni访问Java静态变量的值

|方法名|作用|
|---|---|
|GetStaticFieldId|根据变量名获取target中静态变量的ID|
|GetStaticIntField|根据变量ID获取int静态变量的值，对应的还有byte，boolean，long等|
|SetStaticIntField|修改int静态变量的值，对应的还有byte，boolean，long等|

###### Jni访问Java成员方法

|方法名|作用|
|---|---|
|GetMethodID|根据方法名获取target中成员方法的ID|
|CallVoidMethod|执行无返回值成员方法|
|CallIntMethod|执行int返回值成员方法，对应的还有byte，boolean，long等|

###### Jni访问Java静态方法

|方法名|作用|
|---|---|
|GetStaticMethodID|根据方法名获取target中静态方法的ID|
|CallStaticVoidMethod|执行无返回值静态方法|
|CallStaticIntMethod|执行int返回值静态方法，对应的还有byte，boolean，long等|

###### Jni访问Java构造方法

|方法名|作用|
|---|---|
|GetMethodID|根据方法名获取target中构造方法的ID，注意，方法名传`<init>`|
|NewObject|创建对象|

###### Jni创建引用

|方法名|作用|
|---|---|
|NewGlobalRef|创建全局引用|
|NewWeakGlobalRef|创建弱全局引用|
|NewLocalRef|创建局部引用|
|DeleteGlobalRef|释放全局对象，引用不主动释放会导致内存泄漏|
|DeleteLocalRef|释放局部对象，引用不主动释放会导致内存泄漏|

###### 异常处理机制

|方法名|作用|
|---|---|
|ExceptionOccurred|判断是否有异常发生|
|ExceptionClear|清除异常|
|Throw|往上(java层)抛出异常|
|ThrowNew|往上(java层)抛出自定义异常|

### 实战
---
重要
 * 尽管java中的jniTest()方法没有参数，但cpp中仍然有两个参数，
 * 参数一：**JNIEnv* env表示指向可用JNI函数表的接口指针**，所有跟jni相关的操作都需要通过env来完成
 * 参数二：jobject是调用该方法的java对象，这里是MainActivity调用的，所以thiz代表MainActivity
 * 方法名：**Java_包名_类名_方法名**

```c
#include <jni.h>
#include <string>
#include <android/log.h>

#define TAG    "jasonwan" // 这个是自定义的LOG的标识
#define LOGD(...)  __android_log_print(ANDROID_LOG_DEBUG,TAG,__VA_ARGS__) // 定义LOGD类型

extern "C"
JNIEXPORT void JNICALL
Java_com_jason_jni_MainActivity_jniTest(JNIEnv *env, jobject thiz) {
    //获取MainActivity的class对象
    jclass clazz = env->GetObjectClass(thiz);
    //获取MainActivity中num变量id
    /**
    参数1：MainActivity的class对象
    参数2：变量名称
    参数3：变量类型，具体见上《表3-方法签名》
    **/
    jfieldID numFieldId = env->GetFieldID(clazz, "num", "I");
    //根据变量id获取num的值
    jint oldValue = env->GetIntField(thiz, numFieldId);
    //将num变量的值+1
    env->SetIntField(thiz, numFieldId, oldValue + 1);
    //重新获取num的值
    jint num = env->GetIntField(thiz, numFieldId);
    //先获取tv变量id
    jfieldID tvFieldId = env->GetFieldID(clazz, "tv", "Landroid/widget/TextView;");
    //根据变量id获取textview对象
    jobject tvObject = env->GetObjectField(thiz, tvFieldId);
    //获取textview的class对象
    jclass tvClass = env->GetObjectClass(tvObject);
    //获取setText方法ID
    /**
    参数1：textview的class对象
    参数2：方法名称
    参数3：方法参数类型和返回值类型，具体见上《表3-方法签名》
    **/
    jmethodID methodId = env->GetMethodID(tvClass, "setText", "([CII)V");
    //获取setText所需的参数
    //先将num转化为jstring
    char buf[64];
    sprintf(buf, "%d", num);
    jstring pJstring = env->NewStringUTF(buf);
    const char *value = env->GetStringUTFChars(pJstring, JNI_FALSE);
    //创建char数组，长度为字符串num的长度
    jcharArray charArray = env->NewCharArray(strlen(value));
    //开辟jchar内存空间
    jchar *pArray = (jchar *) calloc(strlen(value), sizeof(jchar));
    //将num字符串缓冲到内存空间中
    for (int i = 0; i < strlen(value); ++i) {
        *(pArray + i) = *(value + i);
    }
    //将缓冲的值写入到上面创建的char数组中
    env->SetCharArrayRegion(charArray, 0, strlen(value), pArray);
    //调用setText方法
    env->CallVoidMethod(tvObject, methodId, charArray, 0, env->GetArrayLength(charArray));
    //释放资源
    env->ReleaseCharArrayElements(charArray, env->GetCharArrayElements(charArray, JNI_FALSE), 0);
    free(pArray);
    pArray = NULL;
}
```

通过这样一个简单的案例，将大部分jni相关的API都练习了一遍，不难看出，java层能实现的功能，在native层一样可以实现，但这里仅仅是为了练习jni，实际项目中不会把一些无关紧要的功能写在native层，比如UI操作，因为同样的功能，java代码要简洁得太多。

上面我们在实现`jniTest()`时，可以看到c++里面的方法名很长`Java_com_jason_jni_MainActivity_jniTest`，这是**jni静态注册**的方式，按照jni规范的命名规则进行查找，格式为`Java_类路径_方法名`，这种方式在应用层开发用的比较广泛，因为Android Studio默认就是用这种方式，而**在framework当中几乎都是采用动态注册的方式来实现java和c/c++的通信**。比如之前研究过的[《Android MediaPlayer源码分析》](https://juejin.cn/post/7166215760321183781 "https://juejin.cn/post/7166215760321183781")，里面就是采用的动态注册的方式。

在Android中，当程序在Java层运行`System.loadLibrary("jnitest");`这行代码后，程序会去载入`libjnitset.so`文件。于此同时，产生一个Load事件，这个事件触发后，程序默认会在载入的`.so`文件的函数列表中查找`JNI_OnLoad`函数并执行，与Load事件相对，在载入的.so文件被卸载时，Unload事件被触发。此时，程序默认会去载入的.so文件的函数列表中查找`JNI_OnLoad`函数并执行，然后卸载.so文件。**因此开发者经常会在`JNI_OnLoad`中做一些初始化操作，动态注册就是在这里进行的**，使用`env->RegisterNatives(clazz, gMethods, numMethods)`

`JNINativeMethod`是jni中定义的一个结构体
```c
typedef struct {
    const char* name; //java中要注册的native方法名
    const char* signature;//方法签名
    void*       fnPtr;//对应映射到C/C++中的函数指针
} JNINativeMethod;
```

相比静态注册，动态注册的灵活性更高，如果修改了native函数所在类的包名或类名，仅调整native函数的签名信息即可。上述案例改为动态注册，java代码不需要更改，只需要更改native代码

```c
#include <jni.h>
#include <string>
#include <android/log.h>

#define TAG    "jasonwan" // 这个是自定义的LOG的标识
#define LOGD(...)  __android_log_print(ANDROID_LOG_DEBUG,TAG,__VA_ARGS__) // 定义LOGD类型

void native_jniTest(JNIEnv *env, jobject thiz) {
    //获取java类的实例对象
    jclass clazz = env->GetObjectClass(thiz);
    //获取MainActivity中num变量
    jfieldID numFieldId = env->GetFieldID(clazz, "num", "I");
    jint oldValue = env->GetIntField(thiz, numFieldId);
    //将num变量的值+1
    env->SetIntField(thiz, numFieldId, oldValue + 1);
    //重新获取num
    jint num = env->GetIntField(thiz, numFieldId);
    //获取tv控件对象
    jfieldID tvFieldId = env->GetFieldID(clazz, "tv", "Landroid/widget/TextView;");
    jobject tvObject = env->GetObjectField(thiz, tvFieldId);
    jclass tvClass = env->GetObjectClass(tvObject);
    //获取setText方法ID
    jmethodID methodId = env->GetMethodID(tvClass, "setText", "([CII)V");
    //获取setText所需的参数
    //先将num转化为jstring
    char buf[64];
    sprintf(buf, "%d", num);
    jstring pJstring = env->NewStringUTF(buf);
    const char *value = env->GetStringUTFChars(pJstring, JNI_FALSE);
    //创建char数组，长度为字符串num的长度
    jcharArray charArray = env->NewCharArray(strlen(value));
    //开辟jchar内存空间
    jchar *pArray = (jchar *) calloc(strlen(value), sizeof(jchar));
    //将num的值缓冲到内存空间中
    for (int i = 0; i < strlen(value); ++i) {
        *(pArray + i) = *(value + i);
    }
    //将缓冲的值写入到char数组中
    env->SetCharArrayRegion(charArray, 0, strlen(value), pArray);
    //调用setText方法
    env->CallVoidMethod(tvObject, methodId, charArray, 0, env->GetArrayLength(charArray));
    //释放资源
    env->ReleaseCharArrayElements(charArray, env->GetCharArrayElements(charArray, JNI_FALSE), 0);
    free(pArray);
    pArray = NULL;
}

static const JNINativeMethod nativeMethod[] = {
    /*
    参数1：java中要注册的native方法名
    参数2：方法签名
    参数3：对应映射到C/C++中的函数指针
    */
        {"jniTest", "()V", (void *) native_jniTest},
};

//System.loadLibrary()执行时会调用此方法
extern "C"
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM *vm, void *reversed) {
    JNIEnv *env = NULL;
    // 初始化JNIEnv
    if (vm->GetEnv((void **) &env, JNI_VERSION_1_4) != JNI_OK) {
        return JNI_FALSE;
    }
    // 找到需要动态注册的java类
    jclass jniClass = env->FindClass("com/jason/jni/MainActivity");
    if (nullptr == jniClass) {
        return JNI_FALSE;
    }
    // 动态注册
    if (env->RegisterNatives(jniClass, nativeMethod, sizeof(nativeMethod) / sizeof(nativeMethod[0])) != JNI_OK) {
        return JNI_FALSE;
    }
    // 返回JNI使用的版本
    return JNI_VERSION_1_4;
}

```

注意，在Android工程中要排除对native方法以及所在类的混淆（java工程不需要），否则要注册的java类和java函数会找不到。

