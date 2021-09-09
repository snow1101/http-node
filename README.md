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
