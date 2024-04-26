typedef关键字的作用是可以用于给数据类型定义一个别名

### typedef创建结构体别名
---
当你定义了一个结构体时，每次创建一个结构体都要使用struct+结构体名的方式，而用了typedef之后，只要一个结构体别名就可以创建了。

```c
struct app_configure {                   //本名
	unsigned char name;
	unsigned char size;
	unsigned char color;
};

struct app_configure app1; 				//必须使用struct
struct app_configure app2;

typedef struct app_configure {			
	unsigned char name;
	unsigned char size;
	unsigned char color;
} App_Config;							//别名

App_Config app1;						//不需要用struct
App_config app2; 
```

### typedef创建数据类型的别名
我们知道C语言定义数据类型的时候只定义了它们之间的关系，但却没有具体定义它们的大小。比如 short 的长度只规定了不大于 int，long的长度不小于 int，int是多大也没确定，所以你会看到51单片机的int大小为两个字节，而在stm32中的长度为 4 字节。所以这个时候有必要使用一个别名来代替具体的数据类型，并且最好这个别名有一定的说明性：

### 注意
---
#### const 
在和const一起使用的时候，本想定义一个指向的字符为常量的变量指针，但因为typedef的特殊性，不是简单的替换，所以最终的定义的是指向的字符为变量的常量指针。

```c
typedef char* pCHAR;				//定义char指针的别名
int func(const pCHAR, const pCHAR);	//想定义: 指针为变量，指向的字符为常量
									//实际： 指针为常量，指向的字符为变量
```

解决的办法就是在typedef中加const即可：

```c
typedef const char* pCHAR;			//定义char指针的别名
int func( pCHAR,  pCHAR);			//指针为变量，指向的字符为常量
```

### 不能与 auto extern static register 一起使用
---
typedef 不影响对象的存储特性，但是在语法上它却是一个存储类的关键字，就像 auto、extern、static、register等关键字一样。所以不能和存储类的关键字一起使用：
```c
typedef extern	 char* pCHAR;   // error
typedef static 	 char* pCHAR;   // error
typedef auto  	 char* pCHAR;   // error
typedef register char* pCHAR;   // error
```

