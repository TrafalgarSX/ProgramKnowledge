之前一直以为 long 的字节数是随着编译位数变化的， 64位下 long 为 8字节， 32位下 long 为 4字节。

实际上并不完全是。

windows平台：
```
32:
int bytes 4
unsigned int bytes 4
long bytes 4
long long bytes 8
pointer bytes 4

64:
int bytes 4
unsigned int bytes 4
long bytes 4
long long bytes 8
pointer bytes 8
```

windows下 不论 32位还是64位， long 都是4字节

linux平台：
```
32:
int bytes is : 4
unsigned int bytes is : 4
long bytes is : 4
long long bytes is : 8
pointer bytes is : 4

64:
int bytes is : 4
unsigned int bytes is : 4
long bytes is : 8
long long bytes is : 8
pointer bytes is : 8
```

![[type_size.png]]


例如在32位环境下long类型通常占4个字节，外字节对齐默认是4；而64位环境下long类型通常占8个字节，外字节对齐默认是8。即使同位数但不同的系统可能也不一样, e.g.
- 64位的Windows系统下long类型通常占4个字节，
- 64位的Mac OS系统下long类型却占8个字节
- 其它类Unix系统下的long类型也是占8个字节，例如Linux


![[编译器数据模型.png]]

