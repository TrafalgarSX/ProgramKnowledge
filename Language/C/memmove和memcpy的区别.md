### memmove和memcpy的作用
memcpy和memmove（）都是C语言中的库函数，在头文件string.h中，作用是拷贝一定长度的内存的内容

他们的作用是一样的，唯一的区别是，当内存发生局部重叠的时候，memmove保证拷贝的结果是正确的，memcpy不保证拷贝的结果的正确的。
![[Pasted image 20230324000047.png]]
1. 这种拷贝重叠的情况不会出现问题。因为重叠的部分已经copy过了，不再需要了。

![[Pasted image 20230323235715.png]]
2. 这种copy重叠的情况会出现问题。因为重叠的部分还没有copy过就被前面的copy给覆盖了，后续重叠部分copy的部分是错误的内容。

memcpy的实现
```c
void *memcpy(void *dst, void *src, size_t count)
{
	assert(NULL != dst || NULL != src);

	char *tmp = (char *) dst;
	char *p = (char *)src;

	do{
		*tmp++ = *p++
	}while(--count);

	return dst;
}
```

memmove的实现
```c
void *memmove(void *dst, void *src, size_t len)
{
	assert(NULL != dst || NULL != src);

	if(dst < src)
	if(dst < src || dst > src + len)
	{
		char *tmp = (char *) dst;
		char *p = (char *)src;
		do{
			*tmp++ = *p++
		}while(--len);
	}else
	{
		//从后往前进行copy
		char *tmp = (char *) dst + len;
		char *p = (char *) src + len;
		do{
			*--tmp = *--p
		}while(--len);
	}

	return dst;
}
```

