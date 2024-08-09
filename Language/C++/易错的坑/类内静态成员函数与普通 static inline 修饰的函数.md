
### 普通函数中的 `static inline`

在头文件中定义的 `static inline` 函数，**每个编译单元都会有一个实例**。这是因为 `inline` 函数的定义通常会被编译器内联展开，而 [`static`](vscode-file://vscode-app/c:/Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html "src/staticlib.cpp") 修饰符则限制了函数的链接范围，使其在每个编译单元中都是独立的。

```cpp
// header.h

static inline void my_function() {

    // function body

}

```
### 类中的静态成员函数

类中的静态成员函数，即使在头文件中定义，也不会在每个编译单元中生成多个实例。**静态成员函数属于类本身，而不是类的某个实例，因此它们在整个程序中只有一个实例。**

```cpp
// header.h

class single {

public:

    static single &get_instance() {

        static single instance;

        return instance;

    }

private:

    single() = default;

    single(const single &) = delete;

    single(single &&) = delete;

    single &operator=(const single &) = delete;

    single &operator=(single &&) = delete;

};
```
### 总结

- **普通函数中的 `static inline`**：每个编译单元都有一个实例，因为 [`static`](vscode-file://vscode-app/c:/Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html "src/staticlib.cpp") 限制了链接范围，而 `inline` 允许在多个编译单元中定义。
- **类中的静态成员函数**：在整个程序中只有一个实例，因为静态成员函数属于类本身，而不是类的某个实例。

note: 不能因为看起来定义连起来是一样的，就认为各种表现都是一样的， 要根据上下文来判断

因此，虽然 `static inline` 修饰的普通函数和类中的静态成员函数在某些方面看起来相似，但它们在编译和链接时的行为是不同的。