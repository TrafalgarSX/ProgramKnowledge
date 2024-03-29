**静态库**：在链接步骤中，链接器将**从库文件取得所需的代码，复制到生成的可执行文件中**，这种库称为静态库，其特点是可执行文件中**包含了库代码的一份完整拷贝**；缺点就是被多次使用就会有多份冗余拷贝。即静态库中的指令都全部被直接包含在最终生成的 EXE 文件中了。

**动态库**：动态链接库是一个包含**可由多个程序同时使用的代码和数据的库**，DLL不是可执行文件。动态链接提供了一种方法，使进程可以调用不属于其可执行代码的函数。函数的可执行代码位于一个 DLL 中，该 DLL 包含一个或多个已被编译、链接并与使用它们的进程分开存储的函数。

👉在VS中新建生成动态库的工程，编译成功后，**产生一个.lib文件和一个.dll文件**。

#### 静态库中的lib和动态库中的lib有什么区别？
**静态库中的lib**：该LIB包含**函数代码本身（即包括函数的索引，也包括实现）**，在编译时直接将代码加入程序当中

**动态库中的lib**：该LIB包含了**函数所在的DLL文件和文件中函数位置的信息（索引）**，函数实现代码由运行时加载在进程空间中的DLL提供

💡总之， lib是编译时用到的， dll是运行时用到的， 如果要完成源代码的编译，只需要lib, 如果要是动态链接的程序运行起来，**只需要dll**。

### 调用动态库
---
有两种方式调用动态库， 一种隐式链接，一种显示链接。

#### 隐式链接
隐式链接 需要.h文件，dll文件，lib文件

（1）将dll放到工程的工作目录

（2）设置项目属性--vc++目录--库目录为lib所在的路径

（3）将lib添加到项目属性--链接器--输入--附加依赖项（或者直接在源代码中加入#pragma comment(lib, “some.lib”)）
💡附加依赖项里面只能放lib, 无论是静态库lib, 还是动态库lib

（4）在源文件中添加.h头文件

#### 显示链接
显示链接 只需要.dll文件，但是这种调用方式不能调用dll中的变量或者类（其实可以调用类，但是相当麻烦，有兴趣者可参考[http://blog.csdn.net/jdcb2001/article/details/1394883](http://blog.csdn.net/jdcb2001/article/details/1394883)）

显示调用主要使用WIN32 API函数LoadLibrary、GetProcAddress，举例如下：
```c++
typedef int (*dllfun)(void);//定义指向dll中函数的函数指针
    HINSTANCE hlib = LoadLibrary(".\\dll.dll");
    if(!hlib)
    {
        std::cout<<"load dll error\n";
        return -1;
    }
    dllfun fun = (dllfun)GetProcAddress(hlib,"fndll");
    if(!fun)
    {
        std::cout<<"load fun error\n";
        return -1;
    }
    fun();
```

💡
DLL 事实上和 EXE 文件一样，同属 PE 格式的执行文件。对于隐式的引用外部符号，需要把外部符号所在的位置写在 PE 头上。PE 加载器将从 PE 头上找到依赖的符号表，并加载依赖的其它 DLL 文件。

unix 上并非如此，so 文件大多为 elf 执行文件格式。当它们需要的外部符号，可以不写明这些符号所在的位置。也就是说，通常 so 文件本身并不知道它依赖的那些符号在哪些 so 里面。这些符号是由调用 dlopen 的进程运行时提供的。而 unix 下的执行文件本身会暴露自己静态链接的符号，（可以是自己本身实现的，或者是从静态库 .a 文件里链入的）。dlopen 将把这些符号通报给 dlopen 加载的 .so 文件，最终完成动态链接。（事实上 dlopen 还可以指定 mode ，完成更复杂的操作）

动态链接 CRT ，静态链接 CRT ，多线程库，单线程库

### 调用静态库
---
调用静态库需要静态库和头文件就可以了。