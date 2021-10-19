# http-node http 学习笔记

### Connection头部 Connection仅针对当前连接有效

* Connection: keep-alive **长连接**  HTTP/1.1默认支持 客户端发送表示请求长连接 服务端发送表示支持长连接
* Connection: close **关闭长链接**  不论request还是response的header中包含了值为close的connection，都表明当前正在使用的tcp链接在请求处理完毕后会被断掉。以后client再进行新的请求时就必须创建新的tcp链接了
* Connection: cookie 等 表示对代理服务器的要求，即，告诉代理服务器不要转发Connection 后面的这些字段， 如Connection: cookie 就是告诉代理服务器，在转发这个请求给原服务器的时候去除cookie


### 与代理相关的头部
因为客户端和服务端，中间或许会有很多代理，每个代理都会建立自己的tcp连接，所以如果服务器想拿到客户端的ip 就要客户端发送相关的请求头部
![image](https://user-images.githubusercontent.com/8045533/132653962-be31b6db-0763-45af-85b2-e4499e5a59bc.png)

* Max-Forwards: 5 通过 TRACE 方法或 OPTIONS 方法，发送含有首部字段 Max-Forwards 的请求时，该字段以十进制整数形式指定可经过的服务器最大数目。[可参考](https://www.bookstack.cn/read/http-study/27.md)
* Via 指明经过的代理服务器名称及版本  当客户端请求到达第一个代理服务器时，该服务器会在自己发出的请求里面添加 Via 头部，并填上自己的相关信息，当下一个代理服务器 收到第一个代理服务器的请求时，会在自己发出的请求里面复制前一个代理服务器的请求的Via 头部，并把自己的相关信息加到后面， 以此类推，当 OCS 收到最后一个代理服务器的请求时，检查 Via 头部，就知道该请求所经过的路由。例如：Via：1.0 236-81.D07071953.sina.com.cn:80 (squid/2.6.STABLE13)[可参考](https://www.bookstack.cn/read/http-study/11.md)
* Cache-Control：no-transform 代表禁止代理服务器修改响应包体

### 验证器响应头部  
* Etag: 给出当前资源表述的标签 分为强验证和弱验证 ， 举个例子 强验证 Etag: "xyzzy" 弱验证  Etag: W/"xyzzy"  加上W/ 就代表弱验证器
* Last-Modified: date

### 条件请求头部
* if-match : etag
* if-none-match: etag
* if-modified-since: date
* if-unmodified-since: date
* if-range: etag/date

### 缓存相关
1. 判断缓存是否过期 —— 按优先级，取以下响应头部的值  **s-maxage > max-age > Expires >  预估过期时间**
   例如： Cache-Control: s-maxage=3600    Cache-Control: max-age=86400   Expires: Fri,03 May 2020 03:15:20 GMT 
   **预估过期时间** 是指现在网络上很多资源是没有返回缓存相关的一些头部的，但是一些资源比如图片又不是经常会变得，所以大多数浏览器根据 RFC7234推荐的算法 （DownloadTime-LastModified）* 10% 作为预估的过期时间
### 描述请求上下文的头部

* user-agent：**请求头**中指明客户端的类型信息，服务器可以根据此对资源的表述做抉择 例如user-agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/93.0.4577.63 Safari/537.36  
* referer：referer是浏览器对来自某一页面的请求自动添加的头部，**请求头**中告诉了这个请求的来源， 对防盗链和防止恶意请求 有一定的作用。比如我只允许我自己的网站访问我自己的图片服务器，那我的域名是www.xxx.com，那么图片服务器每次取到Referer来判断一下是不是我自己的域名www.xxxx.com，如果是就继续访问，不是就拦截。　 Refere不可靠，但有部分作用，起码增加了爬图难度和成本，因为在浏览器端你是无法指定(伪造)，你只能通过定义meta元素告诉浏览器怎么用 <meta name="referrer" content="never"> 
 content有如下值：

1. 如果 referer-policy 的值为 never：删除 http head 中的 referer；
2. 如果 referer-policy 的值为 default：如果当前页面使用的是 https 协议，而正要加载资源使用的是普通的 http 协议，则将 http header 中额 referer 置为空；
3. 如果 referer-policy 的值 origin：只发送 origin 部分；
4. 如果 referer-policy 的值为 always：不改变 http header 中的 referer 的值；

* from: **请求头**主要用于网络爬虫，告诉服务器如何通过邮件联系到爬虫的负责人。 因为邮件地址涉及到隐私信息，所以请求携带From头需要征得用户的同意。RFC协议建议所有的机器人代理发起的请求应该携带此头部，以免遇到问题时可以找到责任人。
* server: **响应头**中指明服务器上所用软件的信息，用于帮助客户端定位问题或者统计数据 例如：Server : Apache/2.2.17 (Unix)
* allow: **响应头**中告诉客户端，服务器上该URL对应的资源允许哪些方法的执行 例如 allow: GET
* accept-ranges: **响应头**中告诉客户端，服务器上该资源是否允许range请求 。当浏览器发现 Accept-Ranges 头时，可以尝试继续中断了的下载，而不是重新开始。 Accept-Ranges: bytes Accept-Ranges: none

### http包体相关的头部

* content-length: 1*digit **发送http消息时，已经可以确定包体的全部长度**，此时返回content-length 并用十进制表明包体中的**字节**个数，且必须与实际传输的包体长度一致，如果表述的长度 小于 实际的包体长度， 包体会被截断。 如果大于包体的实际长度，会导致请求失败！
* transfer-encoding: chunk **发送http消息时，不能够确定包体的全部长度** ， 含有transfer-encoding头部后content-length 头部会被忽略。 由于 Content-Length 字段必须真实反映实体长度，但是对于动态生成的内容来说，在内容创建完之前，长度是不可知的。这时候要想准确获取长度，只能开一个足够大的 buffer，等内容全部生成好再计算。但这样做一方面需要更大的内存开销，另一方面也会让客户端等更久。我们需要一个新的机制：不依赖头部的长度信息，也能知道实体的边界——分块编码（Transfer-Encoding: chunked）
具体方法
  1. 在头部加入 Transfer-Encoding: chunked 之后，就代表这个报文采用了分块编码。这时，报文中的实体需要改为用一系列分块来传输。
  2. 每个分块包含十六进制的长度值和数据，长度值独占一行，长度不包括它结尾的 CRLF(\r\n)，也不包括分块数据结尾的 CRLF。
  3. 最后一个分块长度值必须为 0，对应的分块数据没有内容，表示实体结束。
例如
```
HTTP/1.1 200 OK
Content-Type: text/plain
Transfer-Encoding: chunked

   

25\r\n
This is the data in the first chunk\r\n

   

1C\r\n
and this is the second one\r\n

   

3\r\n

con\r\n

   

8\r\n
sequence\r\n

   

0\r\n

\r\n
```
**Content-Encoding 和 Transfer-Encoding 二者经常会结合来用，其实就是针对 Transfer-Encoding 的分块再进行 Content-Encoding压缩。**
* Content-Type: 实体头部用于指示资源的MIME类型 主类型有 text/image/audio/video/application 等，细分下去可参考[Content-Type对照表](https://tool.oschina.net/commons/)，特别需要注意的一个是在表单提交，并且有一些上传文件的操作时，请求头中Content-Type大多数为multipart/form-data ，这个时候请求头中会有一些Content-Disposition 相关的信息用于表述表单的字段，但是一般在network下看不到，可以通过抓包看具体的响应头部 例如
  ```
  Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW

  ----WebKitFormBoundary7MA4YWxkTrZu0gW
  Content-Disposition: form-data; name="topic"

  xx
  ----WebKitFormBoundary7MA4YWxkTrZu0gW
  Content-Disposition: form-data; name="logLine"
  ```
* Content-Disposition: inline |attachment ｜form-data  Content-Disposition作为响应头指示回复的内容该以何种形式展示，是以内联的形式（即网页或者页面的一部分），还是以附件的形式下载并保存到本地。具体参考[Content-Disposition细分](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Disposition)。这个值中我们需要注意的是 若值为attachment， 则响应直接作为附件下载，比如百度图片，点击下载图标，会发现请求的响应头为Content-Disposition: attachment; filename="3f0c8bea924f46e03545254cbbf6d433.jpeg"  filename的值就是附件的名字
## 状态码 可参考[http状态码大全](https://www.php.cn/web/web-http.html)

* **300** multiple choices  当请求头部带有响应式协商的内容， 服务器会返回300 multiple choices 由客户端户端选择一种标书URI使用  或者406 Not Acceptable
![image](https://user-images.githubusercontent.com/8045533/132810836-945fb7e6-e866-46de-ae4d-ceb0e3bf00b9.png)
* **412** Precondition Failed 客户端错误响应代码指示对目标资源的访问已被拒绝。这种情况与上比其他方法条件请求GET或者HEAD当被定义的条件If-Unmodified-Since或If-None-Match头部不被满足。在这种情况下，无法进行请求（通常是上载或修改资源），并且发送此错误响应。
  ![image](https://user-images.githubusercontent.com/8045533/133038914-e291ec9a-dc90-4325-bd8b-521f600fbf39.png)
* **416** Requested Range Not Satisfiable  如果请求中包含了 Range 请求头，并且 Range 中指定的任何数据范围都与当前资源的可用范围不重合，同时请求中又没有定义 If-Range 请求头，那么服务器就应当返回416状态码。假如 Range 使用的是字节范围，那么这种情况就是指请求指定的所有数据范围的首字节位置都超过了当前资源的长度。服务器也应当在返回416状态码的同时，包含一个 Content-Range 实体头，用以指明当前资源的长度。这个响应也被禁止使用 multipart/byteranges 作为其 Content-Type。
* **206** Partial Content表示该服务器已经成功处理了部分 GET 请求。类似于 FlashGet 或者迅雷这类的 HTTP下载工具都是使用此类响应实现断点续传或者将一个大文档分解为多个下载段同时下载。该请求必须包含 Range 头信息来指示客户端希望得到的内容范围，并且可能包含 If-Range 来作为请求条件。请求头和响应头可以随便打开百度视频，快进到一个点查看请求和响应报文。但是**如果服务器不支持Range请求，则会以200做状态码并返回完整的包体**
![image](https://user-images.githubusercontent.com/8045533/133041908-46624044-b695-4ef0-9d61-712e40ee0174.png)
![image](https://user-images.githubusercontent.com/8045533/133042050-d0f3a67e-369b-4255-8c25-51192cb7f1d5.png)
![image](https://user-images.githubusercontent.com/8045533/133042176-4990f0ef-dae6-4360-a33f-0a404b7e62b8.png)
