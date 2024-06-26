C++类中成员变量的初始化有两种方式：

- 构造函数初始化列表
- 构造函数体内赋值

**初始化列表**对成员变量做的事情叫做**初始化**，而***构造函数体内对成员变量做的事情就叫做赋值**了。

❗初始化与赋值不同。

整个构造函数的执行过程包括（初始化列表以及构造函数体内）都属于**该对象初始化的过程**

初始化早于赋值，它是随着对象的诞生一起进行的。而赋值是在对象诞生以后又给予它一个新的值。

❓TODO 因为初始化列表只调用一次拷贝构造函数。 *8这里调用的究竟是什么构造函数，有的情况下调用的是构造函数，有的情况调用的中复制构造函数。需要确认**

而**构造函数内赋值调用一次默认的构造函数和赋值操作**，效率明显就变差了。

### 构造函数初始化列表
---
构造函数初始化列表以一个冒号开始，接着是以逗号分隔的数据成员列表，每个数据成员后面跟一个放在括号中的初始化式。*变量初始化顺序与列表无关, 与类中成员变量声明的顺序有关。*

#### 类的初始化效率

```cpp
class Student
{
public:
        // 1. 初始化列表进行初始化name
	Student(int id):m_Name("Hello")
	{
		m_Id = id;
	}
private:
	int m_Id;
	std::string m_Name;
};

class Student
{
public:
        // 2. name 赋值 进行初始化
	Student(int id)
	{
		m_Id = id;
                m_Name= "Hello";
	}
private:
	int m_Id;
	std::string m_Name;
};
```

name也是一个类，如果没有在显示的初始化列表中初始化，那么它将调用默认的构造函数初始化，再在构造函数中赋值，**这样就造成了2次调用构造函数**，所以效率并不高。

💡建议在类的初始化列表中进行初始化。



