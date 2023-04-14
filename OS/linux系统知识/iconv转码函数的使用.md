### iconv库的介绍
[ICONV](https://www.gnu.org/software/libiconv/documentation/libiconv-1.17/iconv.3.html)
Linux下的iconv开发库包括iconv_open，iconv_close，iconv等C函数，可以用来在C/C++程序中很方便的​转换字符编码

#### iconv_open
```c
#include <iconv.h>
iconv_t iconv_open (const char* tocode, const char* fromcode);
```
​功能：​为字符集转换分配描述符，该描述符适用于从字符编码fromcode转换为tocode
返回值：
- 成功：​返回一个新分配的转换描述符
- 失败：​设置errno并返回(iconv_t)-1 

- 空的编码名称""等价于“char”，它表示与语言环境有关的字符编码
- 当tocode以字符串“//TRANSLTI”结尾时，音译被激活，意味着当一个字符不能在目标字符集中表示时，可以通过一个或几个看起来与原始字符相似的字符来近似表示
- 当tocode以字符串“//IGNORE”结尾时，目标字符集中无法表示的字符将被默认丢弃

支持的编码：
常用 ASCII ISO-8859-{1,2,3,4,......16 } GBK CP936 GB18030 BIG5-HKSCS 
UTF-8  UCS-2 UCS-2BE UCS-2LE UCS4-4 UCS-4BE UCS-4LE UTF-16{BE, LE} UTF-32{BE, LE}

#### iconv
```c
#include <iconv.h>
size_t iconv(iconv_t cd, char **inbuf, size_t *inbytesleft, char **outbuf,
            size_t *outbytesleft);
```
iconv函数用于转换，参数为iconv_open 产生的转换描述符、输入字符串、输入字符串的长度、输出字符串、输出字符串的长度。由于该函数中，参数的值都要发生变化，所以都是传递的指针。

**重点** 在转换的过程中，inbuf和outbuf的指针会逐渐的向后偏移​，因此inbuf和outbuf指针会改变，因此在操作iconv()之前最后定义两个临时指针分别指向inbuf和outbuf，然后使用这两个临时指针来调用iconv()函数

1. ​当inbuf或outbuf为NULL时函数的表现
```
​如果inbuf为NULL或*inbuf为NULL，但是outbuf不为NULL和*outbuf不为NULL​。在这种情况下，iconv函数会尝试将cd的转换状态设置为初始状态，并将相应的移位序列存储在*outbuf处。 从*outbuf开始，最多将写入*outbytesleft个字节。如果输出缓冲区没有更多空间可用于此复位序列，则会将errno设置为E2BIG并返回(size_t)-1。否则，它将增加*outbuf并减少*outbytesleft 通过写入的字节
```

2. ​如果inbuf为NULL或*inbuf为NULL，且outbuf为NULL或*outbuf为NULL​。在这种情况下，iconv函数会将cd的转换状态设置为初始状态

返回值
- 成功：​返回转换的所有字符中​不可逆转换的字符数​，可逆的不计算在内
- 失败：​设置errno并返回(size_t)-1

#### iconv_close()
```c
#include <iconv.h>
int iconv_close (iconv_t cd);
```
功能：​关闭参数所指定的转换描述符

返回值
- 成功：​返回0
- ​失败：​设置errno并返回-1

返利程序
```c
//utf8_to_utf16.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <iconv.h>

int main()
{
    unsigned char *encTo = "UTF-16";  //要转换的编码格式
    unsigned char *encFrom = "UTF-8"; //原编码格式

    //新建一个转换分配描述符
    iconv_t cd = iconv_open(encTo, encFrom);
    if(cd == (iconv_t)-1)
    {
        perror("iconv_open");
        return -1;
    }

    //转换的字符串
    unsigned char inbuf[1024] = {0}
    size_t inbufLen = strlen(inbuf);

    //用来保存转换后的字符串
    size_t outbufLen = 1024;
    unsigned char outbuf[outbufLen];
    memset(outbuf, 0, outbufLen);

    //因为iconv()函数会改变inbuf和outbuf指针,因此实现定义两个临时指针指向于这两,然后用这两个指针去操作
    unsigned char *inbufSrc = inbuf;
    unsigned char *outbufSrc = outbuf;

    //转换之前打印一下原字符串和每个字符串的二进制值
    printf("inbuf: %s, len: %ld\n", inbuf, strlen(inbuf));
    printf("utf8:\n");
    for(int i = 0; i < inbufLen; ++i)
{
printf("%02x ", inbuf[i]);
}
printf("\n\n\n");


//转换
size_t ret = iconv(cd, (char**)&inbufSrc, &inbufLen, (char**)&outbufSrc, &outbufLen);
if(ret == -1)
{
perror("iconv");
iconv_close(cd);
return -1;
}


//打印转换之后的字符串和每个字符串的二进制值(因为outbuf中间会有0，所以不能简单的使用strlen来计算长度)
printf("outbuf: %s len: %ld\n", outbuf, outbufSrc - outbuf);
printf("utf16:\n");
//不能以strlen(outbuf)作为参数2,因为outbuf中间有0，会导致for循环只打印部分的内容
for(int i = 0; i < (int)(outbufSrc - outbuf); ++i)
    {
        printf("%02x ", outbuf[i]);
    }
    printf("\n");

    //关闭句柄
    iconv_close(cd);

    return 0;
}

```

### 补充：windows下如何进行编码的转换？
