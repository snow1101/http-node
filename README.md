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

* content-length: 1*digit 发送http消息时，已经可以确定包体的全部长度，此时返回content-length 并用十进制表明包体中的**字节**个数，且必须与实际传输的包体长度一致，如果表述的长度 小于 实际的包体长度， 包体会被截断。 如果大于包体的实际长度，会导致请求失败！

## 状态码

* 300 multiple choices  当请求头部带有响应式协商的内容， 服务器会返回300 multiple choices 由客户端户端选择一种标书URI使用  或者406 Not Acceptable
![image](https://user-images.githubusercontent.com/8045533/132810836-945fb7e6-e866-46de-ae4d-ceb0e3bf00b9.png)

