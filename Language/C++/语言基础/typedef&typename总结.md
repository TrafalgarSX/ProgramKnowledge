
用过 C++ 的同学对 typename 和 typedef 相信并不是很陌生，但是当我看到下面这段代码的时候仍然无法理解：

```cpp
typedef typename std::vector<T>::size_type size_type;
```

按理来说 typedef 一般不是用来定义一种类型的别名，如下：

```cpp
typedef int SpeedType;
```

定义了一个 int 的别名是 SpeedType，那么我就可以这样用：

```cpp
int main(void)
{
    SpeedType s = 10;
    printf("speed is %d m/s",s);
    return 0;
}
```

但是 typedef 后面接 typename 表示什么意思呢？typename 不是用来定义模板参数的吗？下面我们分别归纳一下 typedef & typename 的用法。

## **typedef**

首先来看看 typedef 的几种常见用法。

### **为特定含义的类型取别名**

这个我在上面已经讲过了，但是它是定义一种类型的别名，而不只是简单的宏替换，也可以用作同时声明指针型的多个对象的。

比如：

```cpp
char* pa, pb;
cout << typeid(pa).name() << endl; //Pc
cout << typeid(pb).name() << endl; //c
```

本来想要把 pa、pb 两个变量都声明为字符串，但是这样只能成功声明了一个。但是我们使用 typedef 就可以成功声明两个：

```cpp
typedef char* PCHAR;
PCHAR pa, pb; // 可行
cout << typeid(pa).name() << endl; //Pc
cout << typeid(pb).name() << endl; //Pc
```

### **为结构体取别名**

在声明变量的时候，需要带上 struct，即像下面这样使用：

```cpp
typedef struct info
{
    char name[128];
    int length;
}Info;

Info var;
```

### **用来定义与平台无关的类型**

比如定义一个叫 REAL 的浮点类型，在目标平台一上，让它表示最高精度的类型为：

```cpp
typedef long double REAL;
```

在不支持 long double 的平台二上，改为：

```cpp
typedef double REAL;
```

当跨平台时，只要改下 typedef 本身就行，不用对其他源码做任何修改。

## **typename**

typename 关键字用于引入一个模板参数，这个关键字用于指出模板声明（或定义）中的非独立名称（dependent names）是类型名，而非变量名：

```cpp
template <typename T>
const T& max(const T& x, const T& y)
{
  if (y < x) {
    return x;
  }
  return y;
}
```

typename 在这里的意思表明 T 是一个类型。如果没有它的话，在某些情况下会出现模棱两可的情况，比如下面这种情况：

```cpp
template <class T>
void foo() {
    T::iterator * iter;
    // ...

}
```

作者想定义一个指针`iter`，它指向的类型是包含在类作用域`T`中的`iterator`。可能存在这样一个包含`iterator`类型的结构：

```cpp
struct ContainsAType {
    struct iterator { /*...*/ };
};
```

那么 `foo<ContainsAType>();` 这样用的是时候确实可以知道 `iter`是一个`ContainsAType::iterator`类型的指针。但是`T::iterator`实际上可以是以下三种中的任何一种类型：

- 静态数据成员
- 静态成员函数
- 嵌套类型

所以如果是下面这样的情况：

```cpp
struct ContainsAnotherType {
    static int iterator;
    // ...
};
```

那 `T::iterator * iter;`被编译器实例化为`ContainsAnotherType::iterator * iter;`，变成了一个静态数据成员乘以 iter ，这样编译器会找不到另一个变量 `iter` 的定义 。所以为了避免这样的歧义，我们加上 typename，表示 `T::iterator` 一定要是个类型才行。

```cpp
template <class T>
void foo() {
    typename T::iterator * iter;
    // ...
}
```

## **得出结论**

我们回到一开始的例子，对于 `vector::size_type`，我们可以知道：

```cpp
template <class T,class Alloc=alloc>
class vector{
public:
    //...
    typedef size_t size_type;
    //...
};
```

`vector::size_type`是`vector`的嵌套类型定义，其实际等价于 `size_t`类型。

```cpp
typedef typename std::vector<T>::size_type size_type;
```

那么这个例子的真是面目是，`typedef`创建了存在类型的别名，而`typename`告诉编译器`std::vector<T>::size_type`是一个类型而不是一个成员。

C++ typedef & typename知识点总结 - 知乎](https://zhuanlan.zhihu.com/p/617664673)