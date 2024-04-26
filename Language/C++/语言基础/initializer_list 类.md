提供访问元素数组的权限，其中数组的每个成员均具有指定的类型。

```c++
template <class Type>
class initializer_list
```

**Type**:
要在 `initializer_list` 中存储的元素数据类型。

### note 
---
使用大括号内的初始值设定项列表可构造 `initializer_list`。
```c++
initializer_list<int> i1{ 1, 2, 3, 4 };
```

每当函数签名需要 `initializer_list` 时，**编译器将具有同类元素的大括号内的初始值设定项列表转换为 `initializer_list`**。 有关使用 `initializer_list` 的详细信息，请参阅[统一初始化和委托构造函数](https://learn.microsoft.com/zh-cn/cpp/cpp/initializing-classes-and-structs-without-constructors-cpp?view=msvc-170)

### demo
---
```c++
// initializer_list_class.cpp
// compile with: /EHsc
#include <initializer_list>
#include <iostream>

int main()
{
    using namespace std;
    // Create an empty initializer_list c0
    initializer_list <int> c0;

    // Create an initializer_list c1 with 1 element
    initializer_list <int> c1{ 3 };

    // Create an initializer_list c2 with 5 elements
    initializer_list <int> c2{ 5, 4, 3, 2, 1 };

    // Create a copy, initializer_list c3, of initializer_list c2
    initializer_list <int> c3(c2);

    // Create a initializer_list c4 by copying the range c3[ first,  last)
    const int* c3_ptr = c3.begin();
    c3_ptr++;
    c3_ptr++;
    initializer_list <int> c4(c3.begin(), c3_ptr);

    // Move initializer_list c4 to initializer_list c5
    initializer_list <int> c5(move(c4));

    cout << "c1 =";
    for (auto c : c1)
        cout << " " << c;
    cout << endl;

    cout << "c2 =";
    for (auto c : c2)
        cout << " " << c;
    cout << endl;

    cout << "c3 =";
    for (auto c : c3)
        cout << " " << c;
    cout << endl;

    cout << "c5 =";
    for (auto c : c5)
        cout << " " << c;
    cout << endl;
}
```