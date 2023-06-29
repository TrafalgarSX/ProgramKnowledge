### libcurl如何验证证书
---
```c
#include <stdio.h>
#include <curl/curl.h>

int main(void) {
    CURL *curl;
    CURLcode res;
    
    curl_global_init(CURL_GLOBAL_DEFAULT);
    
    curl = curl_easy_init();
    if(curl) {
        // 设置目标URL
        curl_easy_setopt(curl, CURLOPT_URL, "https://www.example.com");
        
        // 启用SSL验证
        curl_easy_setopt(curl, CURLOPT_SSL_VERIFYPEER, 1L);
        
        // 设置CA证书路径
        curl_easy_setopt(curl, CURLOPT_CAINFO, "/path/to/ca_certificate.crt");
        
        // 设置证书类型，默认为PEM
        curl_easy_setopt(curl, CURLOPT_SSLCERTTYPE, "PEM");
        
        // 设置证书文件路径, 这是客户端证书， 一般不会用到，只有双向认证的时候会用到
        curl_easy_setopt(curl, CURLOPT_SSLCERT, "/path/to/client_certificate.crt");
        
        // 设置私钥文件路径
        curl_easy_setopt(curl, CURLOPT_SSLKEY, "/path/to/client_private_key.key");
        
        // 设置私钥密码（如果有）
        curl_easy_setopt(curl, CURLOPT_KEYPASSWD, "password");
        
        // 发送请求
        res = curl_easy_perform(curl);
        
        // 检查请求是否成功
        if(res != CURLE_OK) {
            fprintf(stderr, "curl_easy_perform() failed: %s\n", curl_easy_strerror(res));
        }
        
        // 清理cURL句柄
        curl_easy_cleanup(curl);
    }
    
    // 全局清理
    curl_global_cleanup();
    
    return 0;
}
```

libcurl 启用https会默认验证证书， 但是也可以选择不验证证书：
```c
        curl_easy_setopt(curl, CURLOPT_SSL_VERIFYPEER, 0L);
        curl_easy_setopt(curl, CURLOPT_SSL_VERIFYHOST, 0L);
```

像上面那样设置就可以不验证证书，要注意两个都设置为0，才能真的不验证证书， google搜索相关的结果只说设置第一个就可以，实际上是错误的。

第二个选项是验证域名的， 对于自签名证书，可以选择只验证证书，不验证域名，因为自签名的证书并没有相关的域名字段（可能是这样, 我不是很了解， 经过测试，实践上是这样的）


### SSL 认证
[[SSL认证]]


### libcurl 实现SSL认证
与 SSL 单向认证相关的 `curl_easy_setopt` 选项有以下几个：

- CURLOPT_SSL_VERIFYPEER： cURL 是否验证对等证书（peer's certificate），值为 1，则验证，为 0 则不验证。要验证的交换证书可以在 CURLOPT_CAINFO 选项中设置，或在 CURLOPT_CAPATH中设置证书目录。
- CURLOPT_SSL_VERIFYHOST：值为1 ： cURL 检查服务器SSL证书中**是否存在一个公用名(common name)**；值为2： cURL 会检查公用名是否存在，并且是否与提供的主机名匹配；0 为不检查名称。这里的 common name 是在创建证书过程中指定，例如 subj 选项值中的 `/CN` 值； `openssl req -subj "/C=CN/ST=IL/L=ShenZhen/O=Tencent/OU=Tencent/CN=luffichen_server.tencent.com/emailAddress=luffichen@www.tencent.com" ...`
- CURLOPT_CAINFO：**一个保存着1个或多个用来让服务端验证的证书的文件名**。这个参数仅仅在和CURLOPT_SSL_VERIFYPEER一起使用时才有意义。
- CURLOPT_CAPATH：**一个保存着多个CA证书的目录**。这个选项是和CURLOPT_SSL_VERIFYPEER一起使用的。

与 SSL 双向认证相关的 `curl_easy_setopt` 选项有以下几个：

- CURLOPT_SSLCERT：客户端证书路径
- CURLOPT_SSLCERTTYPE：证书的类型。支持的格式有"PEM" (默认值), "DER"和"ENG"。
- CURLOPT_SSLKEY：客户端私钥的文件路径
- CURLOPT_SSLKEYTYPE：客户端私钥类型，支持的私钥类型为"PEM"(默认值)、"DER"和"ENG"。
- CURLOPT_KEYPASSWD：客户端私钥密码，私钥在创建时可以选择加密。

```c
if(!oneway_certification)
{
    // 验证服务器证书有效性
    curl_easy_setopt(m_curl_handler, CURLOPT_SSL_VERIFYPEER, 1);
    // 检验证书中的主机名和你访问的主机名一致
    curl_easy_setopt(m_curl_handler, CURLOPT_SSL_VERIFYHOST, 2);
    // 指定 CA 证书路径
    curl_easy_setopt(m_curl_handler, CURLOPT_CAINFO, m_ca_cert_file.c_str());
}
else
{
    // 不验证服务器证书
    curl_easy_setopt(m_curl_handler, CURLOPT_SSL_VERIFYPEER, 0);
    curl_easy_setopt(m_curl_handler, CURLOPT_SSL_VERIFYHOST, 0);
}

if(!client_cert_file.empty())
{
    // 客户端证书，用于双向认证
    curl_easy_setopt(m_curl_handler, CURLOPT_SSLCERT, client_cert_file.c_str());
}

if(!client_cert_type.empty())
{
    // 客户端证书类型，用于双向认证
    curl_easy_setopt(m_curl_handler, CURLOPT_SSLCERTTYPE, client_cert_type.c_str());
}

if(!private_key.empty())
{
    // 客户端证书私钥，用于双向认证
    curl_easy_setopt(m_curl_handler, CURLOPT_SSLKEY, private_key.c_str());
}

if(!private_key_type.empty())
{
    // 客户端证书私钥类型，用于双向认证
    curl_easy_setopt(m_curl_handler, CURLOPT_SSLKEYTYPE, private_key_type.c_str());
}

if(!private_key_passwd.empty())
{
    // 客户端证书私钥密码
    curl_easy_setopt(m_curl_handler, CURLOPT_KEYPASSWD, private_key_passwd.c_str());
}

```

#### SSL certificate problem, verify that the CA cert is OK

当 CURLOPT_SSL_VERIFYPEER 为 1 时，表示启用了验证访问的服务器合法性，且必须设置 CURLOPT_CAINFO 或 CURLOPT_CAPATH 其中一个，而 CURLOPT_SSL_VERIFYHOST 为 2 时，表示验证 CA 证书中的 common name 是否与访问的服务器域名是否一致。在测试的时候，需要记得为客户端侧机器添加相应的 host 域名 IP 解析，如果直接使用 IP 访问也会报 SSL certificate problem, verify that the CA cert is OK 错误。


#### curl: (60) SSL certificate : unable to get local issuer certificate

问题的原因有很多，这里只列举一二。

在验证服务器证书时，找不到CA证书，如果正确设置了 cainfo 或 capath 参数且 CA 证书已经是 rootCA，依然出错，那么可能是证书生成的时候出错，再重新生成一个；如果 CA 证书由一个中间证书签发，rootCA 签发中间证书，那么如果服务器没有提供中间证书，在验证过程中，openssl 在形成完整的证书链也会报这个错误，所以 `cat intermediate.crt >> domain.crt` 将所有中间证书与rootCA证书捆绑在一起。


### 处理SSL验证结果
处理验证结果：当进行HTTPS请求时，libcurl会自动验证证书链。您可以使用`CURLOPT_SSL_CTX_FUNCTION`选项设置一个回调函数来处理验证结果。

```cpp
c
curl_easy_setopt(curl, CURLOPT_CAINFO, "/path/to/ca-bundle.crt");
```

在回调函数中，您可以检查验证结果并采取相应的操作。例如，您可以检查验证错误码，或者设置额外的验证选项。

```cpp
c
static int sslctx_callback(CURL* curl, void* sslctx, void* userdata) {
    // 获取SSL上下文
    SSL_CTX* ctx = (SSL_CTX*)sslctx;

    // 获取验证结果
    long verifyresult = SSL_get_verify_result(ssl);

    // 处理验证结果
    if (verifyresult != X509_V_OK) {
        // 验证失败
        fprintf(stderr, "Certificate verification failed: %s\n", X509_verify_cert_error_string(verifyresult));
        // 返回非零值表示验证失败
        return 1;
    }

    // 验证成功
    return 0;
}
```