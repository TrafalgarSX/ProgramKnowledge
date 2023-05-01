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
When `calloc` multiplies `count * size`, it checks for overflow, and errors out if the multiplication returns a value that can't fit into a 32 or 64bit integer.

### Difference 2: lies, damned lies, and virtual memory
```bash
~$ gcc calloc-1GiB-demo.c -o calloc-1GiB-demo
~$ ./calloc-1GiB-demo
calloc+free 1 GiB: 3.44 ms
malloc+memset+free 1 GiB: 365.00 ms
```

i.e. `calloc` is more than 100x faster. Our textbooks and manual pages says they're equivalent. What the heck is going on?

The answer, of course, is That `calloc` is cheating.

For small allocations, `calloc` literally will just call `malloc+memset`, so it'll be the same speed. But for **larger allocations**, most memory allocators will for various reasons **make a special request to the operating system** to fetch more memory just for this alloction.
>"Small" and "large" here are determined by some heuristics inside your memory allocator; for glibc "large" is anything >128 KiB

**When the operating system hands out memory to a process, it always zeros it out first**, because otherwise our process would be able to peek at whatever detritus was left in that memory by the last process to use it, which might include, like, crypto keys, or embarrassing fanfiction. 

1. So that's the first way that calloc cheats: when you call malloc to allocate a large buffer, then probably the memory will come from the operating system and already be zeroed, so there's no need to call memset.

But this only explains part of the speedup: memset+malloc is actually clearing the memory twice, and calloc is clearing it once, so we might expect calloc to be 2x faster at best. Instead... it's 100x faster. What the heck?

It turns out that the kernel is also cheating! When we ask it for 1 GiB of memory, **it doesn't actually go out and find that much RAM and write zeros to it and then hand it to our process. Instead, it fakes it, using virtual memory: it takes a single 4 KiB page of memory that is already full of zeros (which it keeps around for just this purpose)**, and maps 1 GiB / 4 KiB = 262144 copy-on-write copies of it into our process's address space. So the first time we actually write to each of those 262144 pages, then at that point the kernel has to go and **find a real page of RAM, write zeros to it, and then quickly swap it in place of the "virtual" page that was there before.** But this happens lazily, on a page-by-page basis.(也就是说虚拟内存的原因，分配的是页表的代表的内存，每次使用的都是页表中的页，真实使用内存的时候再把页表中调用的页清零后提供给程序使用)

True reason:
So in real life, the difference won't be as stark as it looks in our benchmark up above – **part of the trick is that calloc is shifting some of the cost of zero'ing out pages until later**, while malloc+memset is paying the full price up front. 

