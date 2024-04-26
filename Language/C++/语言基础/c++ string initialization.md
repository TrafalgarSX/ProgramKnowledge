For initialization of a variable definition using assignment, like

```cpp
std::string a = "foo";
```

Here two objects are created: The variable you define (`a` in my example) and a temporary object (for the string `"foo"`). Then the copy-constructor of your variable (`a`) is called with the temporary object, and then the temporary object is destructed. The copy-assignment operator is not called.

However, the copying can be avoided if the compiler uses [copy elision](http://en.wikipedia.org/wiki/Copy_elision), which is an optimization technique to avoid unnecessary copies and copying. The copy-constructor needs to be there anyway, even if it's not called.

---

For the definition

```cpp
std::string a("foo");
```

there is a better [constructor](http://en.cppreference.com/w/cpp/string/basic_string/basic_string) which takes a pointer to a constant string, which the literal `"foo"` can be seen as (string literals are actually constant arrays of `char`, but like all arrays they decay to pointers).