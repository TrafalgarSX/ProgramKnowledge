### new
---
**new operator就是new操作符，不能被重载**，假如A是一个类，那么A * a=new A;实际上执行如下3个过程。
（1）调用operator new分配内存，operator new (sizeof(A))

（2）调用构造函数生成类对象，A::A()

（3）返回相应指针

事实上，分配内存这一操作就是由operator new(size_t)来完成的，**如果类A重载了operator new，那么将调用A::operator new(size_t )，否则调用全局::operator new(size_t )**，后者由C++默认提供。

❗区分开 new operator 和  operator new 

new operator/delete operator 就是 new 和 delete **操作符**
而operator new/operator delete 是**函数。**

- new operator  
	（1）调用**operator new(函数)** 分配足够的空间，并调用相关对象的构造函数  
	（2）不可以被重载
 
- operator new  
	（1）**只分配所要求的空间**，不调用相关对象的构造函数。当无法满足所要求分配的空间时，则  
	        ->如果有new_handler，则调用new_handler，否则  
	        ->如果没要求不抛出异常（以nothrow参数表达），则执行bad_alloc异常，否则  
	        ->返回0  
	（2）可以被重载  
	（3）重载时，**`返回类型必须声明为void*`**
	（4）重载时，第一个参数类型必须为表达要求分配空间的大小（字节），类型为size_t  
	（5）重载时，可以带其它参数

#### operator new

operator new是函数，分为三种形式（前2种不调用构造函数，这点区别于new operator）：  

- void* operator new (std::size_t size) throw (std::bad_alloc);  

- void* operator new (std::size_t size, const std::nothrow_t& nothrow_constant) throw();  

- void* operator new (std::size_t size, void* ptr) throw();  

第一种分配size个字节的存储空间，并将对象类型进行内存对齐。如果成功，返回一个非空的指针指向首地址。失败抛出bad_alloc异常。  

第二种在分配失败时不抛出异常，它返回一个NULL指针。  

第三种是placement new版本，它本质上是对operator new的重载，定义于`#include <new>` 中。**它不分配内存，调用合适的构造函数在ptr所指的地方构造一个对象，之后返回实参指针ptr。**  

~~第一、第二个版本**可以被用户重载，定义自己的版本**，第三种placement new不可重载。 ~~

- A* a = new A; //调用第一种  
- A* a = new(std::nothrow) A; //调用第二种  
- new (p)A(); //调用第三种  

`new (p)A()` 调用placement new之后，还会在p上调用A::A()，这里的p可以是**堆中动态分配的内存，也可以是栈中缓冲。**

下面是重载operator new的一个例子：
```cpp
    #include <iostream>
    #include <string>
    using namespace std;
     
    class X
    {
    public:
    	X()
    	{
    		cout << "X's constructor" << endl;
    	}
    	~X()
    	{
    		cout << "X's destructor" << endl;
    	}
     
     
    	void* operator new(size_t size, string str)
    	{
    		cout << "operator new size " << size << " with string " << str << endl;
    		return ::operator new(size);
    	}
     
     
    	void operator delete(void* pointer)
    	{
    		cout << "operator delete" << endl;
    		::operator delete(pointer);
    	}
    private:
    	int num;
    };
     
    int main()
    {
    	X *p = new("A new class") X;
    	delete p;
    	getchar();
    	return 0;
    }
```

new operator与delete operator的行为是不能够也不应该被改变，这是C++标准作出的承诺。

**而operator new与operator delete和C语言中的malloc与free对应，只负责分配及释放空间**。

**但使用operator new分配的空间必须使用operator delete来释放，而不能使用free**，因为它们对内存使用的登记方式不同。反过来亦是一样。你可以重载operator new和operator delete以实现对内存管理的不同要求，但你不能重载new operator或delete operator以改变它们的行为。

#### 为什么有必要写自己的operator new和operator delete？  
答案通常是：为了效率。缺省的operator new和operator delete具有非常好的通用性，它的这种灵活性也使得在某些特定的场合下，可以进一步改善它的性能。尤其在那些需要动态分配大量的但很小的对象的应用程序里，情况更是如此。具体可参考《Effective C++》中的第二章内存管理。
### placement new
---

placement new 是重载operator new 的一个**标准、全局的版本**，它不能够被自定义的版本代替（不像普通版本的operator new和operator delete能够被替换）。

一般来说，使用new申请空间时，是从系统的“堆”（heap）中分配空间。申请所得的空间的位置是根据当时的内存的实际使用情况决定的。但是，**在某些特殊情况下，可能需要在已分配的特定内存创建对象，这就是所谓的“定位放置new”（placement new）操作。  **

定位放置new操作的语法形式不同于普通的new操作。例如，一般都用如下语句A* p=new A;申请空间，而定位放置new操作则使用如下语句A* p=new (ptr)A;申请空间，其中ptr就是程序员指定的内存首地址。考察如下程序。

```cpp
    #include <iostream>
    using namespace std;
     
    class A
    {
    public:
    	A()
    	{
    		cout << "A's constructor" << endl;
    	}
     
     
    	~A()
    	{
    		cout << "A's destructor" << endl;
    	}
    	
    	void show()
    	{
    		cout << "num:" << num << endl;
    	}
    	
    private:
    	int num;
    };
     
    int main()
    {
    	char mem[100];
    	mem[0] = 'A';
    	mem[1] = '\0';
    	mem[2] = '\0';
    	mem[3] = '\0';
    	cout << (void*)mem << endl;
    	A* p = new (mem)A;
    	cout << p << endl;
    	p->show();
    	p->~A();
    	getchar();
    }
```

（1）用定位放置new操作，既可以在栈(stack)上生成对象，也可以在堆（heap）上生成对象。如本例就是在栈上生成一个对象。  

（2）使用语句`A* p=new (mem) A;` 定位生成对象时，指针p和数组名mem指向同一片存储区。所以，与其说定位放置new操作是利用已经请好的空间，真正的申请空间的工作是在此之前完成的。  

（3）使用语句`A *p=new (mem) A;` 定位生成对象时，会自动调用类A的构造函数，但是由于对象的空间不会自动释放（对象实际上是借用别人的空间），所以必须显示的调用类的析构函数，如本例中的`p->~A()`。  

（4）如果有这样一个场景，我们需要大量的申请一块类似的内存空间，然后又释放掉，比如在在一个server中对于客户端的请求，每个客户端的每一次上行数据我们都需要为此申请一块内存，当我们处理完请求给客户端下行回复时释放掉该内存，表面上看者符合c++的内存管理要求，没有什么错误，但是仔细想想很不合理，**为什么我们每个请求都要重新申请一块内存呢，要知道每一次内从的申请，系统都要在内存中找到一块合适大小的连续的内存空间，这个过程是很慢的（相对而言)，极端情况下，如果当前系统中有大量的内存碎片，并且我们申请的空间很大，甚至有可能失败**。为什么我们不能共用一块我们事先准备好的内存呢？可以的，我们可以使用placement new来构造对象，那么就会在我们指定的内存空间中构造对象。

### new 、operator new 和 placement new 区别

（1）new ：不能被重载，其行为总是一致的。它先调用operator new分配内存，然后调用构造函数初始化那段内存。

new 操作符的执行过程：  
1. 调用operator new分配内存 ；  
2. 调用构造函数生成类对象；  
3. 返回相应指针。

（2）operator new：要实现不同的内存分配行为，应该重载operator new，而不是new。

operator new就像operator + 一样，是可以重载的。如果类中没有重载operator new，那么调用的就是全局的::operator new来完成堆的分配。同理，operator new[]、operator delete、operator delete[]也是可以重载的。

（3）placement new：**只是operator new重载的一个版本**。它并不分配内存，只是返回指向已经分配好的某段内存的一个指针。因此不能删除它，但需要调用对象的析构函数。

如果你想在已经分配的内存中创建一个对象，使用new时行不通的。**也就是说placement new允许你在一个已经分配好的内存中（栈或者堆中）构造一个新的对象**。原型中void* p实际上就是指向一个已经分配好的内存缓冲区的的首地址。


