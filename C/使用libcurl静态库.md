### Windows下使用libcurl静态库
windows下使用libcurl静态库有几点需要注意：
- 需要加宏： CURL_STATICLIB
- 需要额外链接 Ws2_32.lib 和 Wldap32.lib，可能需要Crypt32.lib

### libcurl接口调用方式
libcurl提供了一组C语言API函数直接调用。首先需要提到的两个函数就是`curl_global_init()`和 `curl_global_cleanup()`。libcurl要用到一系列的全局常量，**curl_global_init()函数就是初始化这些变量**，并分配一些全局资源；**curl_global_cleanup()则负责释放这些资源**。因此一般情况下，在调用libcurl函数之前，先用`curl_global_init(CURL_GLOBAL_ALL)`做初始化，在调用完毕后，用`curl_global_cleanup()`退出。

需要注意的是，这些全局变量和资源并不是线程安全的，因此，在多线程应用的环境中，最好不要多次调用`curl_global_init()和curl_global_cleanup()`，调用其他函数并不会改变这些全局变量和资源。

**libcurl支持3种不同的接口调用方式，分别是"easy"、"multi"和"share"模式。**

easy是一组同步接口，函数都是`curl_easy_*`形式，这种模式调用curl_easy_perform()函数进行URL进行数据传输，知道传输完成函数才返回；

multi是一组异步接口，函数都是`curl_multi_*`形式，调用curl_multi_perform()函数进行传输，但是每次调用只传输一片数据，**我们可以用select函数控制多个下载任务进行同步下载，来实现在一个线程中同时下载多个文件；**

multi_handle：libcurl为异步操作提供的接口，**允许调用方在一个线程中处理多个操作（就是easy_handle上的操作，注意是单线程下的）**，内部multi_handle采用堆栈的方式保存多个easy_handle，然后在一个线程中可以同时对多个easy_handle进行处理，multi_handle的执行操作 curl_multi_perform 函数是立即返回的，不会阻塞

share运行在多线程中操作共享数据。

`curl_easy_init()`
此函数需要最先被调用，返回CRUL easy句柄；后续其他函数调用都要用到这个句柄。如果没有调用curl_global_init()，该函数自动调用，但是考虑到线程安全的问题，最好自己调用curl_global_init()。

### 使用libcurl发送 mime格式数据

#### curl_mime_init 创建一个mime句柄
```c
#include<curl/curl.h>
curl_mime *curl_mime_init(CURL *easy_handle);
```

>curl_mime_init创建一个新的空 mime 结构的句柄，该结构旨在与easy_handle一起使用。随后可以使用 mime API 填充此 mime 结构，然后使用curl_easy_setopt调用 中的选项CURLOPT_MIMEPOST附加到easy_handle 。
>使用 mime 句柄是发布 HTTP 表单、格式化并使用 SMTP 发送多部分电子邮件或将此类电子邮件上传到 IMAP 服务器的推荐方法。

simple example:
```c
 CURL *easy = curl_easy_init();
 curl_mime *mime;
 curl_mimepart *part;
 
 /* Build an HTTP form with a single field named "data", */
 mime = curl_mime_init(easy);

// 将新的空白部分附加到mime结构
 part = curl_mime_addpart(mime);

 curl_mime_data(part, "This is the field data", CURL_ZERO_TERMINATED);
// 设置mime部分的名称
 curl_mime_name(part, "data");
 
 /* Post and send it. */
 curl_easy_setopt(easy, CURLOPT_MIMEPOST, mime);
 curl_easy_setopt(easy, CURLOPT_URL, "https://example.com");
 curl_easy_perform(easy);
 
 /* Clean-up. */
 curl_easy_cleanup(easy);
 curl_mime_free(mime);

```

#### curl_mime_name  mime 部分的名称。
这是 HTTP 表单字段的命名方式。name指向以 null 结尾的名称字符串。名称字符串被复制到部件中，因此关联的存储可以在调用后安全地释放或重用。**多次设置一个部件的名称是有效的：只保留最后一次调用设置的值。可以通过将name设置为 NULL 来“取消命名”零件。**

#### curl_mime_data 从内存中设置mime部分的主体数据
数据指向数据字节：那些被复制到部分并且它们的存储可以在调用后安全地重复使用。datasize是数据字节数：可以设置为CURL_ZERO_TERMINATED表示数据是一个以空字符结尾的字符串。

多次设置一个部件的内容是有效的：只保留最后一次调用设置的值。可以通过将data设置为 NULL 来取消分配部分的内容。  
**设置大数据会消耗内存：在这种情况下可以考虑使用curl_mime_data_cb 。**

#### curl_mime_filename 设置mime部分的远程文件名
curl_mime_filename设置 mime 部分的远程文件名。**设置远程文件名时，无论部件的内容源是什么，内容数据都会作为文件处理。** （e.g. 数组也会被当作文件处理）
部件的远程文件名在关联的 Content-Disposition 生成的标头中传输到服务器。

```c
 curl_mime *mime;
 curl_mimepart *part;
 
 /* create a mime handle */
 mime = curl_mime_init(easy);
 
 /* add a part */
 part = curl_mime_addpart(mime);
 
 /* send image data from memory */
 curl_mime_data(part, imagebuf, imagebuf_len);
/* 数组被当作文件处理 */
 
 /* set a file name to make it look like a file upload */
 curl_mime_filename(part, "image.png");
 
 /* set name */
 curl_mime_name(part, "data");

```

####  curl_mime_filedata 从命名文件的内容中设置 mime 部分的正文内容。
这是curl_mime_data的**替代方法**，用于将数据设置为 mime 部分。

在文件传输期间以流式方式读取文件的内容，以允许在不使用太多内存的情况下传输大文件。因此，**它要求文件在整个请求期间保持完整。**
如果在实际读取文件之前无法确定文件大小（例如对于设备或命名管道），则包含该部分的整个 mime 结构将通过 HTTP 作为块传输并被 IMAP 拒绝。

#### curl_mime_type 设置mime部分的内容类型
```c
#include <curl/curl.h>
CURLcode curl_mime_type(curl_mimepart *part, const char *mimetype);
```

在没有 mime 类型的情况下，如果协议规范需要，则默认 mime 类型由上下文确定：
1.如果设置为自定义标头，则使用此值。
2.HTTP 表单发布 application/form-data。
3.如果设置了远程文件名，则 mime 类型取自文件扩展名，默认情况下为 application/octet-stream。
4.对于多部分，multipart/mixed。
5.其他情况下的 text/plain。

