### jdb cheatsheet
```
help                          帮助命令
stop at <class id>.<method>   方法上设置断点
stop at <class id>:<line>     代码行设置断点
run                           运行程序

list        查看代码
next        单步运行
step        进入函数
step up     跳出函数
cont        继续运行到下一个断点

locals      查看当前的所有本地变量
print       查看表达式的值
dump        查看对象的详细信息，例如各个字段的信息

classes                     可以查看已经加载的所有类
class <class id>            查看类信息，包括父类，嵌套类
methods <class id>          查看类的成员函数，包括继承的
fields <class id>           查看类的成员变量

clear                           查看所有断点
clear <class id>:<line>         清除行中断点
clear <class id>.<method>       清除方法中的断点
```

查看byte数组
```
byte [] decompressed = new byte[OUTPUT_FILE_IO_BUFFER_SIZE];

jdb
>> dump decompressed
>> This dumps the byte values!
```