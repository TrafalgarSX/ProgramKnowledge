### JNI(Java Native Interface)
---
JNI 全称为 Java Native Interface，是一种允许 Java 代码与本地（Native）代码交互的技术。JNI 提供了一组 API，可以使 Java 程序访问和调用本地方法和资源，也可以使本地代码访问和调用 Java 对象和方法。 此方案需要使用 JNI 进行双向调用。

![[JNI主要内容.png]]


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
JNI 对于 Java 的基础数据类型（int 等）和引用数据类型（Object、Class、数组等）的处理方式不同。这个原理非常重要，理解这个原理才能理解后面所有 JNI 函数的设计思路：

- **基础数据类型：** 会直接转换为 C/C++ 的基础数据类型，例如 int 类型映射为 jint 类型。由于 jint 是 C/C++ 类型，所以可以直接当作普通 C/C++ 变量使用，而不需要依赖 JNIEnv 环境对象；
- **引用数据类型：** 对象只会转换为一个 C/C++ 指针，例如 Object 类型映射为 jobject 类型。由于指针指向 Java 虚拟机内部的数据结构，所以不可能直接在 C/C++ 代码中操作对象，而是需要依赖 JNIEnv 环境对象。另外，为了避免对象在使用时突然被回收，在本地方法返回前，虚拟机会固定（pin）对象，阻止其 GC。

❗另外需要特别注意一点，基础数据类型在映射时是直接映射，而不会发生数据格式转换。例如，Java `char` 类型在映射为 `jchar` 后旧是保持 Java 层的样子，**数据长度依旧是 2 个字节**，而字符编码依旧是 UNT-16 编码。

jni给我们提供了若干个映射表，将java中的类型与jni中的类型进行了一 一映射，其中包括基本数据类型映射，引用数据类型映射，方法签名(包含参数和返回值)映射，以下是这三个映射表：
基本数据类型映射表
![[基本数据类型.png]]

引用数据类型映射表
![[引用数据类型.png]]

方法签名
![[方法签名.png]]

##### 字符串类型操作 jstring
上面提到 Java 对象会映射为一个 jobject 指针，那么 Java 中的 java.lang.String 字符串类型也会映射为一个 jobject 指针。**可能是因为字符串的使用频率实在是太高了**，所以 JNI 规范还专门定义了一个 jobject 的派生类 `jstring` 来表示 Java String 类型，这个相对特殊。

```cpp
// 内部的数据结构还是看不到，由虚拟机实现
class _jstring : public _jobject {};
typedef _jstring*       jstring;

struct JNINativeInterface {
    // String 转换为 UTF-8 字符串
    const char* (*GetStringUTFChars)(JNIEnv*, jstring, jboolean*);
    // 释放 GetStringUTFChars 生成的 UTF-8 字符串
    void        (*ReleaseStringUTFChars)(JNIEnv*, jstring, const char*);
    // 构造新的 String 字符串
    jstring     (*NewStringUTF)(JNIEnv*, const char*);
    // 获取 String 字符串的长度
    jsize       (*GetStringUTFLength)(JNIEnv*, jstring);
    // 将 String 复制到预分配的 char* 数组中
    void        (*GetStringUTFRegion)(JNIEnv*, jstring, jsize, jsize, char*);
};

```

由于 Java 与 C/C++ 默认使用不同的字符编码，因此在操作字符数据时，需要特别注意在 UTF-16 和 UTF-8 两种编码之间转换。

以下为 2 种较为常见的转换场景：

- **1、Java String 对象转换为 C/C++ 字符串：** 调用 `GetStringUTFChars` 函数将一个 jstring 指针转换为一个 UTF-8 的 C/C++ 字符串，并在不再使用时调用 `ReleaseStringChars` 函数释放内存；
- **2、构造 Java String 对象：** 调用 `NewStringUTF` 函数构造一个新的 Java String 字符串对象。

```cpp
// 示例 1：将 Java String 转换为 C/C++ 字符串
jstring jStr = ...; // Java 层传递过来的 String
const char *str = env->GetStringUTFChars(jStr, JNI_FALSE);
if(!str) {
    // OutOfMemoryError
    return;
}
// 释放 GetStringUTFChars 生成的 UTF-8 字符串
env->ReleaseStringUTFChars(jStr, str);

// 示例 2：构造 Java String 对象（将 C/C++ 字符串转换为 Java String）
jstring newStr = env->NewStringUTF("在 Native 层构造 Java String");
if (newStr) {
    // 通过 JNIEnv 方法将 jstring 调用 Java 方法（jstring 本身就是 Java String 的映射，可以直接传递到 Java 层）
    ...
}
```

此处对 GetStringUTFChars 函数的第 3 个参数 `isCopy` 做解释：它是一个布尔值参数，将决定使用拷贝模式还是复用模式：

- **1、JNI_TRUE：** 使用**拷贝模式**，JVM 将**拷贝一份原始数据**来生成 UTF-8 字符串；
- **2、JNI_FALSE：** 使用复用模式，JVM 将**复用同一份原始数据来**生成 UTF-8 字符串。**复用模式绝不能修改字符串内容**，否则 JVM 中的原始字符串也会被修改，打破 String 不可变性。

⚠️💡
```md
The ReleaseString-Chars call is necessary whether GetStringChars has set *isCopy to JNI_TRUE or JNI_FALSE. ReleaseStringChars either frees the copy or unpins the instance, depending upon whether GetStringChars has returned a copy or not.

```

另外还有一个基于范围的转换函数：`GetStringUTFRegion`：预分配一块字符数组缓冲区，然后将 String 数据复制到这块缓冲区中。**由于这个函数本身不会做任何内存分配，所以不需要调用对应的释放资源函数**，也不会抛出 `OutOfMemoryError`。另外，GetStringUTFRegion 这个函数会做越界检查并抛出 `StringIndexOutOfBoundsException` 异常。

```cpp
jstring jStr = ...; // Java 层传递过来的 String
char outbuf[128];
int len = env->GetStringLength(jStr);
env->GetStringUTFRegion(jStr, 0, len, outbuf);
```

##### 数组类型操作
与 jstring 的处理方式类似，JNI 规范将 Java 数组定义为 jobject 的**派生类** `jarray` ：

- 基础类型数组：定义为 `jbooleanArray` 、`jintArray` 等；
- 引用类型数组：定义为 `jobjectArray` 。

下面区分基础类型数组和引用类型数组两种情况：

###### **操作基础类型数组（以 jintArray 为例）：**

- **1、Java 基本类型数组转换为 C/C++ 数组：** 调用 `GetIntArrayElements` 函数将一个 jintArray 指针转换为 C/C++ int 数组；
- **2、修改 Java 基本类型数组：** 调用 `ReleaseIntArrayElements` 函数并使用模式 0；
- **3、构造 Java 基本类型数组：** 调用 `NewIntArray` 函数构造 Java int 数组。

```cpp
extern "C"
JNIEXPORT jintArray JNICALL
Java_com_xurui_hellojni_HelloWorld_generateIntArray(JNIEnv *env, jobject thiz, jint size) {
    // 新建 Java int[]
    jintArray jarr = env->NewIntArray(size);
    // 转换为 C/C ++ int[]
    int *carr = env->GetIntArrayElements(jarr, JNI_FALSE);
    // 赋值
    for (int i = 0; i < size; i++) {
        carr[i] = i;
    }
    // 释放资源并回写
    env->ReleaseIntArrayElements(jarr, carr, 0);
    // 返回数组
    return jarr;
}
```

此处重点对 ReleaseIntArrayElements 函数的第 3 个参数 `mode` 做解释：它是一个模式参数：

| 参数 mode  | 描述                                                     |
| ---------- | -------------------------------------------------------- |
| 0          | 将 C/C++ 数组的数据回写到 Java 数组，并释放 C/C++ 数组   |
| JNI_COMMIT | 将 C/C++ 数组的数据回写到 Java 数组，并不释放 C/C++ 数组 |
| JNI_ABORT  | 不回写数据，但释放 C/C++ 数组                            |

另外 JNI 还提供了基于范围函数：`GetIntArrayRegion` 和 `SetIntArrayRegion`，使用方法和注意事项和 GetStringUTFRegion 也是类似的，**也是基于一块预分配的数组缓冲区**。

###### **操作引用类型数组（jobjectArray）：**

- **1、将 Java 引用类型数组转换为 C/C++ 数组：** 不支持！与基本类型数组不同，引用类型数组的元素 jobject 是一个指针，不存在转换为 C/C++ 数组的概念；
- **2、修改 Java 引用类型数组：** 调用 `SetObjectArrayElement` 函数修改指定下标元素；
- **3、构造 Java 引用类型数组：** 先调用 `FindClass` 函数获取 Class 对象，再调用 `NewObjectArray` 函数构造对象数组。

```cpp
extern "C"
JNIEXPORT jobjectArray JNICALL
Java_com_xurui_hellojni_HelloWorld_generateStringArray(JNIEnv *env, jobject thiz, jint size) {
    // 获取 String Class
    jclass jStringClazz = env->FindClass("java/lang/String");
    // 初始值（可为空）
    jstring initialStr = env->NewStringUTF("初始值");
    // 创建 Java String[]
    jobjectArray jarr = env->NewObjectArray(size, jStringClazz, initialStr);
    // 赋值
    for (int i = 0; i < size; i++) {
        char str[5];
        sprintf(str, "%d", i);
        jstring jStr = env->NewStringUTF(str);
        env->SetObjectArrayElement(jarr, i, jStr);
    }
    // 返回数组
    return jarr;
}
```

#### 方法调用
接下来就可以在jni方法中访问java成员了，同样的，jni给我们提供了一系列访问java成员的API，具体如下：

#### 字段描述符与方法描述符
在 Java 源码中定义的字段和方法，在编译后都会按照**既定的规则记录在 Class 文件中的字段表和方法表结构中**(字节码的规则)。

例如，一个 public String str; 字段会被拆分为字段访问标记（public）、字段简单名称（str）和字段描述符（Ljava/lang/String）。 **因此，从 JNI 访问 Java 层的字段或方法时，首先就是要获取在 Class 文件中记录的简单名称和描述符。**

[[Java_Class文件结构]]

#### JNI 访问 Java 字段
本地代码访问 Java 字段的流程分为 2 步：

- **1、通过 jclass 获取字段 ID**，例如：`Fid = env->GetFieldId(clz, "name", "Ljava/lang/String;");`
- **2、通过字段 ID 访问字段**，例如：`Jstr = env->GetObjectField(thiz, Fid);`

Java 字段分为静态字段和实例字段，相关方法如下：

- GetFieldId：获取实例方法的字段 ID
- GetStaticFieldId：获取静态方法的字段 ID
- GetField：获取类型为 Type 的实例字段（例如 GetIntField）
- SetField：设置类型为 Type 的实例字段（例如 SetIntField）
- GetStaticField：获取类型为 Type 的静态字段（例如 GetStaticIntField）
- SetStaticField：设置类型为 Type 的静态字段（例如 SetStaticIntField）

```cpp
extern "C"
JNIEXPORT void JNICALL
Java_com_xurui_hellojni_HelloWorld_accessField(JNIEnv *env, jobject thiz) {
    // 获取 jclass
    jclass clz = env->GetObjectClass(thiz);
    // 示例：修改 Java 静态变量值
    // 静态字段 ID
    jfieldID sFieldId = env->GetStaticFieldID(clz, "sName", "Ljava/lang/String;");
    // 访问静态字段
    if (sFieldId) {
        // Java 方法的返回值 String 映射为 jstring
        jstring jStr = static_cast<jstring>(env->GetStaticObjectField(clz, sFieldId));
        // 将 jstring 转换为 C 风格字符串
        const char *sStr = env->GetStringUTFChars(jStr, JNI_FALSE);
        // 释放资源
        env->ReleaseStringUTFChars(jStr, sStr);
        // 构造 jstring
        jstring newStr = env->NewStringUTF("静态字段 - Peng");
        if (newStr) {
            // jstring 本身就是 Java String 的映射，可以直接传递到 Java 层
            env->SetStaticObjectField(clz, sFieldId, newStr);
        }
    }
    // 示例：修改 Java 成员变量值
    // 实例字段 ID
    jfieldID mFieldId = env->GetFieldID(clz, "mName", "Ljava/lang/String;");
    // 访问实例字段
    if (mFieldId) {
        jstring jStr = static_cast<jstring>(env->GetObjectField(thiz, mFieldId));
        // 转换为 C 字符串
        const char *sStr = env->GetStringUTFChars(jStr, JNI_FALSE);
        // 释放资源
        env->ReleaseStringUTFChars(jStr, sStr);
        // 构造 jstring
        jstring newStr = env->NewStringUTF("实例字段 - Peng");
        if (newStr) {
            // jstring 本身就是 Java String 的映射，可以直接传递到 Java 层
            env->SetObjectField(thiz, mFieldId, newStr);
        }
    }
}

```

#### JNI 调用 Java 方法
本地代码访问 Java 方法与访问 Java 字段类似，访问流程分为 2 步：

- **1、通过 jclass 获取「方法 ID」**，例如：`Mid = env->GetMethodID(jclass, "helloJava", "()V");`
- **2、通过方法 ID 调用方法**，例如：`env->CallVoidMethod(thiz, Mid);`

Java 方法分为静态方法和实例方法，相关方法如下：

- GetMethodId：获取实例方法 ID
- GetStaticMethodId：获取静态方法 ID
- CallMethod：调用返回类型为 Type 的实例方法（例如 GetVoidMethod）
- CallStaticMethod：调用返回类型为 Type 的静态方法（例如 CallStaticVoidMethod）
- CallNonvirtualMethod：调用返回类型为 Type 的父类方法（例如 CallNonvirtualVoidMethod）

```cpp
extern "C"
JNIEXPORT void JNICALL
Java_com_xurui_hellojni_HelloWorld_accessMethod(JNIEnv *env, jobject thiz) {
    // 获取 jclass
    jclass clz = env->GetObjectClass(thiz);
    // 示例：调用 Java 静态方法
    // 静态方法 ID
    jmethodID sMethodId = env->GetStaticMethodID(clz, "sHelloJava", "()V");
    if (sMethodId) {
        env->CallStaticVoidMethod(clz, sMethodId);
    }
    // 示例：调用 Java 实例方法
    // 实例方法 ID
    jmethodID mMethodId = env->GetMethodID(clz, "helloJava", "()V");
    if (mMethodId) {
        env->CallVoidMethod(thiz, mMethodId);
    }
}

```



##### Jni各种方法分类
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

### JNI 中的对象引用管理
---
#### Java 和 C/C++ 中对象内存回收区别（重点理解）
在讨论 JNI 中的对象引用管理，我们先回顾一下 Java 和 C/C++ 在对象内存回收上的区别：

- **Java：** 对象在堆 / 方法区上分配，由**垃圾回收器扫描对象可达性进行回收**。如果使用局部变量指向对象，在不再使用对象时可以手动显式置空，也可以等到方法返回时自动隐式置空。如果使用全局变量（static）指向对象，在不再使用对象时必须手动显式置空。
- **C/C++：** 栈上分配的对象会在方法返回时自动回收，而堆上分配的对象不会随着方法返回而回收，也没有垃圾回收器管理，因此必须手动回收（free/delete）。

而 JNI 层作为 Java 层和 C/C++ 层之间的桥接层，那么它就会兼具两者的特点：对于

- **局部 Java 对象引用：** 在 JNI 层可以通过 `NewObject` 等函数创建 Java 对象，并且返回对象的引用，这个引用就是 Local 型的局部引用。
	- 对于局部引用：
		- 可以通过 `DeleteLocalRef` 函数手动显式释放（这类似于在 Java 中显式置空局部变量），
		- 也可以等到**函数返回时自动释放**（这类似于在 Java 中方法返回时隐式置空局部变量）；（只有返回的对象会被自动释放）
  
	❗在 JNI 中，当在 C 层创建 Java 对象时，需要手动管理对象的生命周期。在函数返回之前，可以通过返回对象的引用，将对象传递给 Java 层进行管理，或者通过调用 DeleteLocalRef 或 DeleteGlobalRef 函数来手动释放对象的引用。**如果不释放对象的引用，可能会导致内存泄漏或其他问题。**
    ❗❗❗ 我之前有误解， 以为只有最后返回的 Java引用会自动被 JVM 管理， 不用自己管理， 其他都需要 DeleteLocalRef, 实际上并不是， native method 里所有的本地引用都只在 native method 内有效， 函数返回后， 所有的本地引用都会失效。

	💡如果将创建的对象作为返回值返回给 Java 层，那么在 C 层就不需要手动释放对象的引用了。当返回值传递到 Java 层后，Java 层会负责管理对象的生命周期，包括释放对象的引用和最终的垃圾回收。

   官方文档：
   📒Local references are **valid for the duration of a native method call**. **They are freed automatically after the native method returns**. Each local reference costs some amount of Java Virtual Machine resource. 
   
   Programmers need to make sure that native methods do not excessively allocate local references. Although local references are automatically freed after the native method returns to Java, **excessive allocation of local references may cause the VM to run out of memory during the execution of a native method.**
  
- **全局 Java 对象引用：** 由于局部引用在函数返回后一定会释放，可以通过 `NewGlobalRef` 函数将局部引用升级为 Global 型全局变量，这样就可以在方法使用对象（这类似于在 Java 中使用 static 变量指向对象）。在不再使用对象时必须调用 `DeleteGlobalRef` 函数释放全局引用（这类似于在 Java 中显式置空 static 变量）。

> **提示：** 我们这里所说的 ”置空“ 只是将指向变量的值赋值为 null，而不是回收对象，Java 对象回收是交给垃圾回收器处理的。

[Java Native Interface Specification: 4 - JNI Functions](https://docs.oracle.com/en/java/javase/14/docs/specs/jni/functions.html#local-references)
#### JNI中的三大引用
- **1、局部引用：** 大部分 JNI 函数会创建局部引用，局部引用只有在创建引用的本地方法返回前有效，也只在创建局部引用的线程中有效。**在方法返回后，局部引用会自动释放**，也可以通过 `DeleteLocalRef` 函数手动释放；
- **2、全局引用：** 局部引用要**跨方法和跨线程**必须升级为全局引用，全局引用通过 `NewGlobalRef` 函数创建，不再使用对象时必须通过 `DeleteGlobalRef` 函数释放。
- **3、弱全局引用：** 弱引用与全局引用类似，区别在于弱全局引用不会持有强引用，因此不会阻止垃圾回收器回收引用指向的对象。弱全局引用通过 `NewGlobalWeakRef` 函数创建，**不再使用对象时必须通过 `DeleteGlobalWeakRef` 函数释放。** （弱全局引用有着和全局引用一样的生命周期(跨调用和跨线程), 但是有可能会自动被GC释放。）

💡 可以参考[JNI 局部引用和全局引用 - 掘金](https://juejin.cn/post/6934309297815977992)
[JNI程序规范和指南5——JNI中的局部引用和全局引用 | Soo-Q6](https://soo-q6.github.io/blog/2019-10-02-JNI-guides-and-specifications-5/)

❗局部引用只在创建它的函数返回之前有效，一旦函数返回，局部引用就会被释放。**你不能将局部引用缓存在一个静态变量中以供下一次函数调用时使用**。

```cpp
// 局部引用
jclass localRefClz = env->FindClass("java/lang/String");
env->DeleteLocalRef(localRefClz);

// 全局引用
jclass globalRefClz = env->NewGlobalRef(localRefClz);
env->DeleteGlobalRef(globalRefClz);

// 弱全局引用
jclass weakRefClz = env->NewWeakGlobalRef(localRefClz);
env->DeleteGlobalWeakRef(weakRefClz);

```

#### JNI 引用的实现原理
在 JavaVM 和 JNIEnv 中，会分别建立多个表管理引用：

- **JavaVM** 内有 globals 和 weak_globals 两个表管理全局引用和弱全局引用。由于 JavaVM 是进程共享的，**因此全局引用可以跨方法和跨线程共享**；
- **JavaEnv** 内有 locals 表管理局部引用，由于 JavaEnv 是线程独占的，因此局部引用不能跨线程。另外虚拟机在进入和退出本地方法通过 Cookie 信息记录哪些局部引用是在哪些本地方法中创建的，因此局部引用是不能跨方法的。

#### 比较引用是否指向相同对象
可以使用 JNI 函数 `IsSameObject` 判断两个引用是否指向相同对象（适用于三种引用类型），返回值为 `JNI_TRUE` 时表示相同，返回值为 `JNI_FALSE` 表示不同。例如：

```cpp
jclass localRef = ...
jclass globalRef = ...
bool isSampe = env->IsSamObject(localRef, globalRef）

```

另外，当引用与 `NULL` 比较时含义略有不同：

- **局部引用和全局引用与 NULL 比较：** 用于判断引用是否指向 NULL 对象；
- **弱全局引用与 NULL 比较：** 用于判断引用指向的对象**是否被回收**。

### JNI 中的异常处理
---
#### JNI 的异常处理机制（重点理解）
JNI 中的异常机制与 Java 和 C/C++ 的处理机制都不同：

- **Java 和 C/C++：** 程序使用关键字 `throw` 抛出异常，虚拟机会中断当前执行流程，转而去寻找匹配的 catch{} 块，或者继续向外层抛出寻找匹配 catch {} 块。
- **JNI**： 程序使用 JNI 函数 `ThrowNew` 抛出异常，**程序不会中断当前执行流程**，而是返回 Java 层后，虚拟机才会抛出这个异常。

因此，在 JNI 层出现异常时，有 2 种处理选择：

- **方法 1：** 直接 `return` 当前方法，让 Java 层去处理这个异常（这类似于在 Java 中向方法外层抛出异常）；
- **方法 2：** 通过 JNI 函数 `ExceptionClear` 清除这个异常，再执行异常处理程序（这类似于在 Java 中 try-catch 处理异常）。需要注意的是，当异常发生时，必须先处理-清除异常，再执行其他 JNI 函数调用。 **因为当运行环境存在未处理的异常时，只能调用 2 种 JNI 函数：异常护理函数和清理资源函数。**

JNI 提供了以下与异常处理相关的 JNI 函数：

- **ThrowNew：** 向 Java 层抛出异常；
- **ExceptionDescribe：** 打印异常描述信息；
- **ExceptionOccurred：** 检查当前环境是否发生异常，如果存在异常则返回该异常对象；
- **ExceptionCheck：** 检查当前环境是否发生异常，如果存在异常则返回 JNI_TRUE，否则返回 JNI_FALSE；
- **ExceptionClear：** 清除当前环境的异常。

```cpp
struct JNINativeInterface {
    // 抛出异常
    jint        (*ThrowNew)(JNIEnv *, jclass, const char *);
    // 检查异常
    jthrowable  (*ExceptionOccurred)(JNIEnv*);
    // 检查异常
    jboolean    (*ExceptionCheck)(JNIEnv*);
    // 清除异常
    void        (*ExceptionClear)(JNIEnv*);
};
```

```cpp
// 示例 1：向 Java 层抛出异常
jclass exceptionClz = env->FindClass("java/lang/IllegalArgumentException");
env->ThrowNew(exceptionClz, "来自 Native 的异常");
// 示例 2：检查当前环境是否发生异常（类似于 Java try{}）
jthrowable exc = env->ExceptionOccurred(env);
if(exc) {
    // 处理异常（类似于 Java 的 catch{}）
}
// 示例 3：清除异常
env->ExceptionClear();
```

#### 检查是否发生异常的方式
异常处理的步骤我懂了，由于虚拟机在遇到 ThrowNew 时不会中断当前执行流程，那我怎么知道当前已经发生异常呢？有 2 种方法：

- **方法 1：** 通过函数返回值错误码，大部分 JNI 函数和库函数都会有特定的返回值来标示错误，例如 -1、NULL 等。**在程序流程中可以多检查函数返回值来判断异常**。
- **方法 2：** 通过 JNI 函数 `ExceptionOccurred` 或 `ExceptionCheck` 检查当前是否有异常发生。

### JNI 与多线程
---
#### 不能跨线程的引用
在 JNI 中，有 2 类引用是无法跨线程调用的，必须时刻谨记：

- **JNIEnv：** JNIEnv 只在所在的线程有效，在不同线程中调用 JNI 函数时，必须使用该线程专门的 JNIEnv 指针，不能跨线程传递和使用。通过 `AttachCurrentThread` 函数将当前线程依附到 JavaVM 上，获得属于当前线程的 JNIEnv 指针。如果当前线程已经依附到 JavaVM，也可以直接使用 GetEnv 函数。

```cpp
JNIEnv * env_child;
vm->AttachCurrentThread(&env_child, nullptr);
// 使用 JNIEnv*
vm->DetachCurrentThread();
```

- **局部引用：** 局部引用只在创建的线程和方法中有效，不能跨线程使用。可以将局部引用升级为全局引用后跨线程使用。

```cpp
// 局部引用
jclass localRefClz = env->FindClass("java/lang/String");
// 释放局部引用（非必须）
env->DeleteLocalRef(localRefClz);
// 局部引用升级为全局引用
jclass globalRefClz = env->NewGlobalRef(localRefClz);
// 释放全局引用（必须）
env->DeleteGlobalRef(globalRefClz);
```

#### 监视器同步
在 JNI 中也会存在多个线程同时访问一个内存资源的情况，此时需要保证并发安全。在 Java 中我们会通过 synchronized 关键字来实现互斥块（背后是使用监视器字节码），在 JNI 层也提供了类似效果的 JNI 函数：

- **MonitorEnter：** 进入同步块，如果另一个线程已经进入该 jobject 的监视器，则当前线程会阻塞；
- **MonitorExit：** 退出同步块，如果当前线程未进入该 jobject 的监视器，则会抛出 `IllegalMonitorStateException` 异常。

```cpp
// jni.h
struct JNINativeInterface {
    jint        (*MonitorEnter)(JNIEnv*, jobject);
    jint        (*MonitorExit)(JNIEnv*, jobject);
}
```

```cpp
// 进入监视器
if (env->MonitorEnter(obj) != JNI_OK) {
    // 建立监视器的资源分配不成功等
}

// 此处为同步块
if (env->ExceptionOccurred()) {
    // 必须保证有对应的 MonitorExit，否则可能出现死锁
    if (env->MonitorExit(obj) != JNI_OK) {
        ...
    };
    return;
}

// 退出监视器
if (env->MonitorExit(obj) != JNI_OK) {
    ...
};

```

#### 等待与唤醒
JNI 没有提供 Object 的 wati/notify 相关功能的函数，**需要通过 JNI 调用 Java 方法的方式来实现**：
```cpp
static jmethodID MID_Object_wait;
static jmethodID MID_Object_notify;
static jmethodID MID_Object_notifyAll;

void
JNU_MonitorWait(JNIEnv *env, jobject object, jlong timeout) {
    env->CallVoidMethod(object, MID_Object_wait, timeout);
}
void
JNU_MonitorNotify(JNIEnv *env, jobject object) {
    env->CallVoidMethod(object, MID_Object_notify);
}
void
JNU_MonitorNotifyAll(JNIEnv *env, jobject object) {
    env->CallVoidMethod(object, MID_Object_notifyAll);
}

```

#### 创建线程的方法
在 JNI 开发中，有两种创建线程的方式：

- **方法 1 - 通过 Java API 创建：** 使用我们熟悉的 `Thread#start()` 可以创建线程，优点是可以方便地设置线程名称和调试；
- **方法 2 - 通过 C/C++ API 创建：** 使用 `pthread_create()` 或 `std::thread` 也可以创建线程

```cpp
// 
void *thr_fn(void *arg) {
    printids("new thread: ");
    return NULL;
}

int main(void) {
    pthread_t ntid;
    // 第 4 个参数将传递到 thr_fn 的参数 arg 中
    err = pthread_create(&ntid, NULL, thr_fn, NULL);
    if (err != 0) {
        printf("can't create thread: %s\n", strerror(err));
    }
    return 0;
}
```



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
#### 静态注册
上面我们在实现`jniTest()`时，可以看到c++里面的方法名很长`Java_com_jason_jni_MainActivity_jniTest`，这是**jni静态注册**的方式，按照jni规范的命名规则进行查找，格式为`Java_类路径_方法名`，这种方式在应用层开发用的比较广泛，因为Android Studio默认就是用这种方式，而**在framework当中几乎都是采用动态注册的方式来实现java和c/c++的通信**。比如之前研究过的[《Android MediaPlayer源码分析》](https://juejin.cn/post/7166215760321183781 "https://juejin.cn/post/7166215760321183781")，里面就是采用的动态注册的方式。

#### 动态库加载与卸载 JNI_OnLoad函数， JNI_UnLoad函数
在Android中，当程序在Java层运行`System.loadLibrary("jnitest");`这行代码后，程序会去载入`libjnitset.so`文件。于此同时，产生一个Load事件，这个事件触发后，程序默认会在载入的`.so`文件的函数列表中查找`JNI_OnLoad`函数并执行，与Load事件相对，在载入的.so文件被卸载时，Unload事件被触发。此时，程序默认会去载入的.so文件的函数列表中查找`JNI_OnLoad`函数并执行，然后卸载.so文件。**因此开发者经常会在`JNI_OnLoad`中做一些初始化操作，动态注册就是在这里进行的**，使用`env->RegisterNatives(clazz, gMethods, numMethods)`

`JNINativeMethod`是jni中定义的一个结构体
```c
typedef struct {
    const char* name; //java中要注册的native方法名
    const char* signature;//方法签名
    void*       fnPtr;//对应映射到C/C++中的函数指针
} JNINativeMethod;
```

##### 动态注册的优点
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

### 缓存ID
---
**为什么要缓存 ID**：访问 Java 层字段或方法时，需要先利用**字段名 / 方法名和描述符进行检索**，获得 jfieldID / jmethodID。这个检索过程比较耗时，优化方法是将字段 ID 和方法 ID 缓存起来，减少重复检索。

**缓存 ID 的方法**：缓存字段 ID 和 方法 ID的方法主要有两种：
- 使用时缓存
- 初始化时缓存，

主要区别在于缓存发生的时机和缓存 ID 的时效性。

#### 使用时缓存
使用时缓存是指在首次访问字段或方法时，将字段 ID 或方法 ID 存储在**静态变量**中。这样在将来再次调用本地方法时，就不需要重复检索 ID 了。

💡**提示：** 多个线程访问这个本地方法，会使用相同的缓存 ID，会出现问题吗？不会，多个线程计算的字段 ID 或方法 ID 其实是相同的。

#### 静态初始化时缓存
静态初始化时缓存是指在 Java 类初始化的时候，提前缓存字段 ID 和方法 ID。例如：
```cpp
private static native void initIDs();

static {
        // Java 类初始化
        System.loadLibrary("InstanceMethodCall");
        initIDs();
}
----------------------------------------------------
jmethodID cid;
jmethoidID stringId;

JNIEXPORT void JNICALL
Java_InstanceMethodCall_initIDs(JNIEnv *env, jclass cls) {
    cid = (*env)->GetMethodID(env, cls, "callback", "()V");
    jclass stringClazz = (*env)->FindClass(env,"java/lang/String");
    stringId = (*env)->GetMethodID(env,stringClazz,"<init>","([C)V");
}

```

一开始就缓存，使用全局变量保存

#### 两种缓存 ID 方式的对比和使用场景
在大多数情况下，应该尽可能在静态初始化时缓存字段 ID 和方法 ID，因为**使用时缓存存在一些局限性**：
- 1、每次使用前都要**检查缓存有效**；
- 2、字段 ID 和方法 ID 在 Java 类卸载 (**unload**) 时会失效，因此需要确保类卸载之后不会继续使用这个 ID。而静态初始化时缓存在类加载 (load) 时重新检索 ID，因此不用担心 ID 失效。

当然，使用时缓存也不是一无是处。
- 如果**无法修改 Java 代码源码**，使用时缓存是必然的选择。
- 另一个优势在于，使用时缓存相当于懒初始化，可以按需检索 ID，而静态初始化时缓存相当于提前初始化，会一次性检索所有 ID。尽管如此，大多数情况下还是会使用静态初始化时缓存。

#### 什么是 ID，什么是引用？
- **引用**是通过本地代码来管理 JVM 中的资源，可以同时创建多个引用指向相同对象
- **字段 ID 和方法 ID 由 JVM 管理**，同一个字段或方法的 ID 是固定的，只有在所属类被卸载时会失效。

### 加载 & 卸载 so 库的过程
---
[NDK | 说说 so 库从加载到卸载的全过程 - 掘金](https://juejin.cn/post/6892793299427491854)
简单总结：
- 1、so 库加载到卸载的大体过程，主要分为：**确定 so 库绝对路径、nativeLoad 加载进内存、ClassLoader 卸载时跟随卸载**；
- 2、搜索 so 库的路径，分为 App 路径（`/data/app/[packagename]/lib/arm64`）和系统路径（`/system/lib64、/vendor/lib64`）；
- 3、`JNI_OnLoad`与`JNI_OnUnLoad`分别在 so 库加载与卸载时执行。

![[Android加载so库.png]]

### `JNIEnv *` 和 `JavaVM`
---
#### JNIEnv * 指针的作用
JNIEnv* 指针指向一个 JNI 函数表，在本地代码中，可以通过这些函数来访问 JVM 中的数据结构。从这个意义上说，可以理解为 JNIEnv* **指向了 Java 环境**，但不能说 JNIEnv* 代表 Java 环境。

![[JNIEnv作用.png]]

**需要注意：** 如果本地方法被不同的线程调用，传入的 JNIEnv 指针是不同的。**JNIEnv 指针只在它所在的线程中有效，不能跨线程（甚至跨进程）传递和使用**。但 JNIEnv 间接指向的函数表在多个线程间是共享的。

#### JavaVM 的作用
JavaVM 表示 **Java 虚拟机**，一个 Java 虚拟机对应一个 JavaVM 对象，**这个对象是线程间共享的**。我们可以通过 JNIEnv* 来获取一个 JavaVM 对象：
```cpp
jint GetJavaVM(JNIEnv *env, JavaVM **vm);

- vm：用来存放获得的虚拟机的指针的指针；
- return：成功返回0，失败返回其它。

```

#### 在任意位置获取 JNIEnv* 指针

JNIEnv* 指针仅在创建它的线程有效，如果我们需要在其他线程访问JVM，那么必须先调用 AttachCurrentThread 将当前线程与 JVM 进行关联，然后才能获得JNIEnv* 指针。另外，需要调用DetachCurrentThread 来解除链接。
```cpp
jint AttachCurrentThread(JavaVM* vm , JNIEnv** env , JavaVMAttachArgs* args);

- vm：虚拟机对象指针；
- env：用来保存得到的 JNIEnv 指针的指针；
- args：链接参数，参数结构体如下所示；
- return：链接成功返回 0，连接失败返回其它。
-----------------------------------
func() {
    JNIEnv *env;
    (*jvm)->AttachCurrentThread(jvm, (void **)&env, NULL);
}


```


### 参考
---
[NDK 系列（5）：JNI 从入门到实践，万字爆肝详解！ - 掘金](https://juejin.cn/post/7125338583959306248#heading-23)
