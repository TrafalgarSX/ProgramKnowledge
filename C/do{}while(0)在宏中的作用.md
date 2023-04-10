### 为什么有些宏定义通过do{}while(0)包围起来？
```c
#define __set_task_state(tsk, state_value)      \
    do { (tsk)->state = (state_value); } while (0)
```

上面是一个例子， 里面的宏定义被 do{}while(0)包含起来。

**在Linux内核和其它一些著名的C库中有许多使用do{...}while(0)的宏定义。**这种宏的用途是什么？有什么好处？**
```c
do{...}while(0)在C中是唯一的构造程序，让你定义的宏总是以相同的方式工作，这样不管怎么使用宏（尤其在没有用大括号包围调用宏的语句），宏后面的分号也是相同的效果
```

i.e. 使用do{...}while(0)构造后的宏定义不会受到大括号、分号等的影响，总是会按你期望的方式调用运行。


这里你可能感到迷惑不解了，**_为什么不用大括号`_直接_`把宏包围起来呢？_**为什么非得使用**do/while(0)**逻辑呢？**
e.g.
```c
#define foo(x)  { bar(x); baz(x); }

if(!feral)
    foo(wolf);
else
	bin(wolf);

宏扩展后将变成：
if(!feral){
	bar(wolf);
	baz(wolf);
};
else
	bin(wolf);

此时就有语法错误了。
```


**总结：Linux和其它代码库里的宏都用do/while(0)来包围执行逻辑，因为它能确保宏的行为总是相同的，而不管在调用代码中使用了多少分号和大括号。**
