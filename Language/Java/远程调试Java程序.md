### 远程调试Linux上的Java程序
#### 方法一 使用远程服务器上的jdb调试
jdb是java官方的调试工具，随着jdk一起发布，可以实现基本的调试功能，有了jdb，调试java的时候就告别GDB吧

注意，要顺利调试java程序，java代码编译时必须加上 “-g” 参数
```
javac -g -sourcepath app/src/main/java/ -d app/src/main/target app/src/main/java/test/App.java
```

##### 基本用法
jdb的使用格式为：
```
jdb [ options ] [ class ] [ arguments ]
```
-   option，传递给jdb的参数。详见[https://docs.oracle.com/en/java/javase/16/docs/specs/man/jdb.html#options-for-the-jdb-command](https://docs.oracle.com/en/java/javase/16/docs/specs/man/jdb.html#options-for-the-jdb-command)
-   class，要调试的java入口类
-   arguments，传递给java入口类的参数
⚠️在实战中发现，使用jdb调试的时候，options最好指定sourcepath, classpath参数，否则可能无法显示代码或者其他错误，sourcepath是代码路径（**就是main所在类的路径**），classpath是依赖库或者class文件路径(**就是依赖jar包的路径**)。

假如当前路径为app目录，项目的包是test。目录结构为：
```
app
└── src
    ├── main
    │   ├── java
    │   │   └── test
    │   │       └── App.java
    │   └── target
    │       └── test
    │           └── App.class
```

**那么source路径就是从当前路径到包test上一级**，也就是src/main/java。在app目录下执行：
```
jdb -sourcepath src/main/java/ -classpath src/main/target test.App
```

如果依赖库是jar文件，则要指定jar文件的路径，下面指定调试程序的classpath包含appcation-java/lib下面的所有jar包，多个路径之间用冒号分隔，classpath的参数要放在双引号里面
```
jdb -classpath "application-java/lib/*:src/main/target"  -sourcepath src/main/java/ test.App
```

注意最后要调试的类必须是“包+类”这样的完整名称，即test.App，不能只是App
**💡这里应该最好是 包+类这样完整的名称，否则你可能在调试的时候找不到类，我目前没试过，但是当前用 包加类是完全没问题的**

启动成功后，jdb会停住，等待设置断点，我们在main方法设置个断点:
```
stop at test.App.main
```
**💡这里注意 他设置断点是包括包名一起设置的， 中间用 . 分隔，不是用 /， 只有这样设置的断点才能起作用**

然后开始调试:
```
run
```
[[jdb]]  [[gdb调试和附加进程调试]]

##### Ref
[jdb调试java程序 – 李无知](http://liwuzhi.art/?p=606)
https://docs.oracle.com/javase/7/docs/technotes/tools/windows/jdb.html
https://stackoverflow.com/questions/20018866/specifying-sourcepath-in-jdb-what-am-i-doing-wrong

#### 方法二 使用idea远程调试功能（暂未尝试，有条件限制使用不方便）
⚠️ 前提，保证本地代码和服务器上一致
1. Idea添加一个启动方式： `Edit Configurations`
2. 选择添加远程debug jvm
![[Pasted image 20230329164931.png]]

3. 配置远程服务器地址和调试端口
![[Pasted image 20230329164959.png]]

4. 启动程序时 添加参数（这看起来是调试springboot程序）：
```
java -jar -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005 xxx.jar

```

```
注意：我们选择的是5005端口调试，因为服务器的5005端口要开放  
查看是否有防火墙，以firewalld为例，如果是iptables,请参考iptables的相应命令[]

systemctl status firewalld

如果防火墙关了直接调试，  
如果防火墙开启，则需要开放5005端口
firewall-cmd --permanent --add-port=5005/tcp
#重启防火墙(修改配置后要重启防火墙)
firewall-cmd --reload

```

5. 本机调试，成功后状态，直接开始打断点，界面发请求调试即可。

java -agentlib:jdwp=transport=dt_shmem,address=jdbconn,server=y,suspend=n -cp .:com/eetrust/test/EtCosign.jar com/eetrust/test/Test

jdb -attach jdbconn