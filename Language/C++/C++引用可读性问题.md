### C++代码可读性差， e.g. 引用
---
![[引用可读性差的表现.png]]


按[引用传递](https://www.zhihu.com/search?q=%E5%BC%95%E7%94%A8%E4%BC%A0%E9%80%92&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2666222197%7D)在函数调用时没有特别的标识的确是语言设计的失误。其根本原因是当初搞引用语法的时候，复用了[取地址运算符](https://www.zhihu.com/search?q=%E5%8F%96%E5%9C%B0%E5%9D%80%E8%BF%90%E7%AE%97%E7%AC%A6&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2666222197%7D)。这就导致按照这位作者的想法，**&i和&j事实上变成了取i和j的地址，所以在C++是不能这样写的。**

这个缺陷算是C++早期设计的缺陷之一，但是现在也可以通过IDE来缓解（**因为IDE能识别出来调用的是哪个重载，从而确定是按引用传递还是值传递**）。

说白了就是当年（到今天也是如此），C++委员会的口味一直都是坚持不要加关键字，上下文关键字也不行。所以几乎所有的新特性都只能复用现有的**符号**。

C++坚持不加关键字其实也是其变得如此复杂的原因之一，另一个原因是尽量和C兼容……

在当年可能是必要的坚持，但是在今天看来就是口味有些奇特人为制造障碍。就成了历史包袱……

当然，我的意思是，任何语言都有历史包袱……

  

当年不是没考虑过写成`swap( (int&) i, (int&) j );`……

另外C++的引用和Java的[引用类型](https://www.zhihu.com/search?q=%E5%BC%95%E7%94%A8%E7%B1%BB%E5%9E%8B&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2666222197%7D)也不一样，**C++的引用本质是别名，int& i = j，i是j的别名，两者是一个东西**。

**Java的引用类型本质是指针包装**，object i = j，本质上是将j的指针（引用）赋值给i，i和j仍然是独立变量，可以独立操作……

方法重载简化了代码的编写，但也带来了无穷无尽的阅读问题，这本来在语言发展史上是个月经问题。很多小年轻不知道而已。

当年C++加入函数重载本来带来的深远影响包括ABI兼容性问题一直延续至今。C语言正是因为没有重载所以ABI兼容这事儿解决起来简单得多。

**那就是别名式的引用又不配合immutable特性，用起来容易出问题和别扭是正常的。所以，事实上今天也是const&用的更为广泛……**

### 科普引用知识
---
最后做个小科普，**引用类型和按引用传递以及C++的引用（别名）**。并不是一回事儿：

- 引用类型的特点，引用类型的变量只存储引用而不存储值。**可以理解为引用类型本身就是一个指针**，但是取成员的点运算符（.）被解释为[箭头运算符](https://www.zhihu.com/search?q=%E7%AE%AD%E5%A4%B4%E8%BF%90%E7%AE%97%E7%AC%A6&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2666222197%7D)（->）。所以引用类型可以被重新赋值，譬如说：

```csharp
string a = "A";
string b = a;
b = "B";
```

a的值不会被改变，始终是"A"。**这与C++的引用是完全不同的**。

按引用传递即是图片中的swap的例子，**变量到底是穿透进函数，还是在函数边界进行值拷贝的区别**（注意引用类型的值是指针），**事实上Java只允许按照值传递，压根儿就不支持引用传递**。

```csharp
string a = "A";

void Test( string s )
{
  s = "B";
}

Test( a );
```

上面的代码无论C#还是Java都不能改变a的值。

- C++的引用事实上更类似的是别名，和指针什么的都没啥关系。

int& a = b，本质上是创造了b的一个alias，对a做的任何操作都会反映到b之上，如果不跨越函数边界，编译器直接把a和b视为一个符号就完了……也就是说更类似于typedef。
