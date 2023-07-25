### http工作原理
http请求分为3个部分：**状态行，请求头，请求体**

#### Content-Type 请求头报头域
Content-Type(MediaType),i.g. Internet Media Type,互联网媒体类型，也叫做MIME类型。在互联网中有成百上千种不同的而数据类型，HTTP在传输数据对象时会为他们打上被称为MIME的数据格式标签， 用于区分数据类型。

最初MIME用于电子邮件系统，后来HTTP也采用这一方案。在HTTP协议消息头中，使用Content-Type来表示请求和响应中的媒体类型信息。

```
Content-Type的格式：Content-Type: type/subtype; parameter

type:主类型： 任意的字符串，如text, *代表所有
subtype: 子类型：任意的字符串，如html，用“/”与主类型隔开；*代表所有
parameter:可选参数， 如charset, boundary等

e.g.
Content-Type: text/html;
Content-Type:application/json;charset:utf-8;
```

##### application/x-www-form-urlencoded
HTTP会将请求参数用key1=val1&key2=val2的方式进行组织，并**放到请求实体里面**，注意如果是中文或特殊字符如"/“、”,“、“:” 等会自动进行URL转码。不支持文件，一般用于表单提交。

##### multipart/form-data
与application/x-www-form-urlencoded不同，这是一个多部分多媒体类型。**首先生成了一个 boundary 用于分割不同的字段，在请求实体里每个参数以------boundary开始，然后是附加信息和参数名，然后是空行，最后是参数内容。** 多个参数将会有多个boundary块。如果参数是文件会有特别的文件域。最后以------boundary–为结束标识。multipart/form-data支持文件上传的格式，一般需要上传文件的表单则用该类型。
![[http文件上传#http如何实现文件上传]]

##### application/json
JSON是一种轻量级的数据格式，以键值对的方式组织数据。使用这个类型，要求参数本身就是JSON格式的数据，参数会被直接放到请求实体里，不做任何处理。

### 传输格式


###  Transfer-Encoding: chunked 是什么? 什么是分块传输？