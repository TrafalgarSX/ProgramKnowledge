### Why is conversion from string constant to 'char*' valid in C but invalid in C++
---
Up through C++03, your first example was valid, but used a deprecated implicit conversion--a string literal should be treated as being of type `char const *`, since you can't modify its contents (without causing undefined behavior).

As of C++11, the implicit conversion that had been deprecated was officially removed, so code that depends on it (like your first example) should no longer compile.

You've noted one way to allow the code to compile: although the implicit conversion has been removed, an _explicit_ conversion still works, so you can add a cast. I would _not_, however, consider this "fixing" the code.

Truly fixing the code requires changing the type of the pointer to the correct type:

```csharp
char const *p = "abc"; // valid and safe in either C or C++.
```

As to why it was allowed in C++ (and still is in C): simply because there's a lot of existing code that depends on that implicit conversion, and breaking that code (at least without some official warning) apparently seemed to the standard committees like a bad idea.