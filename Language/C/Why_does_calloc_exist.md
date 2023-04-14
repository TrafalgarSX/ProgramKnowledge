When programming in C, there are two standard ways to allocate some new memory on the heap:
```c
void* buffer1 = malloc(size);
void* buffer2 = calloc(count, size);
```

`malloc`allocates an **uninitialized** array with the given number of bytes, i.e, `buffer1`.
`calloc` is different in two ways:
- first, it takes two arguments instead of one
- second, it returns memory that is pre-initialized to be all-zeros.

Lots of book claim that `calloc` call above is equivalent to calling `malloc` and then calling `memset` to fill the memory with zeros:
```c
void* buffer3 = malloc(count* size);
memset(buffer3, 0, count * size);
```

So... why does `calloc` exist?

### Difference 1: computers are bad at arithmetic

### Difference 2: lies, damned lies, and virtual memory