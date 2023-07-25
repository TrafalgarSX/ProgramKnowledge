作者：彼岸  
链接：https://www.zhihu.com/question/451327108/answer/2421548523  
来源：知乎  
-   为啥要找新工作？在现在这家公司呆了三年多了，基本对现在的业务非常熟悉了，已经积累了足够的经验和能力，平时感觉有点温水泡青蛙了，想换个新平台挑战一下。
-   我看你简历上有博客，我问问你写的，什么是多态？多态一般来说指的是基类指针或[引用调用](https://www.zhihu.com/search?q=%E5%BC%95%E7%94%A8%E8%B0%83%E7%94%A8&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2421548523%7D)成员方法时，能够调用到实际类型的成员方法。（我接着扩展了一下背后原理[虚函数表](https://www.zhihu.com/search?q=%E8%99%9A%E5%87%BD%E6%95%B0%E8%A1%A8&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2421548523%7D)和虚[函数指针](https://www.zhihu.com/search?q=%E5%87%BD%E6%95%B0%E6%8C%87%E9%92%88&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2421548523%7D)）
-   多态除了用指针还可以用什么？引用。
-   [纯虚函数](https://www.zhihu.com/search?q=%E7%BA%AF%E8%99%9A%E5%87%BD%E6%95%B0&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2421548523%7D)和虚函数有什么区别？纯虚函数指的是标记为0的函数，这种函数所在的类不能被实例化，而虚函数就没有这个限制。
-   多态还有一种叫编译时多态，有没有了解？了解，就是[函数重载](https://www.zhihu.com/search?q=%E5%87%BD%E6%95%B0%E9%87%8D%E8%BD%BD&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2421548523%7D)嘛，函数名一样，参数个数或类型或返回值不一样。“返回值不一样也是函数重载吗？”我思索片刻说”返回值不一样应该不算，因为编译器无法区分调哪个函数了，刚刚我说错了。“
-   构造函数和[析构函数](https://www.zhihu.com/search?q=%E6%9E%90%E6%9E%84%E5%87%BD%E6%95%B0&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2421548523%7D)可不可以是虚函数？析构函数可以。构造函数我不确定，我好像没见过把构造函数标记为虚函数的。
-   析构函数标记为虚函数，有什么意义？为了派生类对象析构时能够调用到基类的析构。接着问“那析构函数不标记为虚函数，析构时也可以调用到基类的析构方法啊”，“啊我不太确定，我对此理解得不够透彻，平时很少用多态”，“那你工作用什么比较多？”，“用[单例模式](https://www.zhihu.com/search?q=%E5%8D%95%E4%BE%8B%E6%A8%A1%E5%BC%8F&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2421548523%7D)比较多”。
-   Qt和QML熟不熟？Qt比较熟悉，QML很少用了。
-   你说说[信号与槽](https://www.zhihu.com/search?q=%E4%BF%A1%E5%8F%B7%E4%B8%8E%E6%A7%BD&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2421548523%7D)？一个对象发射信号，对应的另外一个对象的[槽函数](https://www.zhihu.com/search?q=%E6%A7%BD%E5%87%BD%E6%95%B0&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2421548523%7D)就会被执行。
-   信号可不可以连接到信号？可以的。
-   信号与槽有哪几种连接方式？我笑了笑说，就第5个参数嘛，我记得是有4,5种，但是现在我只能想起2种，一种是直接连接，相当于直接调用，另外一种是队列的方式，背后会有事件循环，循环到了才会去调用。
-   多线程了不了解？了解，经常用。
-   [线程同步](https://www.zhihu.com/search?q=%E7%BA%BF%E7%A8%8B%E5%90%8C%E6%AD%A5&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2421548523%7D)原语有哪些？锁，互斥量，条件变量，信号量。
-   互斥量和信号量有什么区别？特例，当信号量只有0和1时，就退化为互斥量。（聊到这里我举了一个常见的多线程模型：生产者和消费者模式，还大力夸了一下C++11，说C++11多线程跨平台用得相当方便）


    # this is heuristically generated, and may not be correct
    find_package(protobuf CONFIG REQUIRED)
    target_link_libraries(main PRIVATE protobuf::libprotoc protobuf::libprotobuf protobuf::libprotobuf-lite)

./vcpkg.exe help triplet


    find_package(glog CONFIG REQUIRED)
    target_link_libraries(main PRIVATE glog::glog)