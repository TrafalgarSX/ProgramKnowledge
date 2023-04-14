#### time_t
```c
time_t time(time_t *ptr);
```
为什么它既把时间写进 `*ptr`，又作为[返回值](https://www.zhihu.com/search?q=%E8%BF%94%E5%9B%9E%E5%80%BC&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A281822565%7D)返回呢？见过有人一本正经地论证这个设计有多么牛逼…… 其实哪有那么复杂啊，就是因为 time_t 最初是一个[结构体](https://www.zhihu.com/search?q=%E7%BB%93%E6%9E%84%E4%BD%93&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A281822565%7D)，当时是 `void time(time_t *[ptr])`，后来改成整数了，加上返回值方便一点，又为减少对旧代码的影响没有删掉那个参数。

### & | 优先级
& | 优先级比 == 低，不要试图去认证它有多合理，它就是不合理。这么设计，纯粹是因为最早的 C 没有 && ||，当时逻辑与、或也用 & |（对，没有[短路特性](https://www.zhihu.com/search?q=%E7%9F%AD%E8%B7%AF%E7%89%B9%E6%80%A7&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A281822565%7D)）遗留下来的。

