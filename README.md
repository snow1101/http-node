# http-node http 学习笔记

### Connection头部 Connection仅针对当前连接有效

* Connection: keep-alive **长连接**  HTTP/1.1默认支持 客户端发送表示请求长连接 服务端发送表示支持长连接
* Connection: close **关闭长链接**  不论request还是response的header中包含了值为close的connection，都表明当前正在使用的tcp链接在请求处理完毕后会被断掉。以后client再进行新的请求时就必须创建新的tcp链接了
* Connection: cookie 等 表示对代理服务器的要求，即，告诉代理服务器不要转发Connection 后面的这些字段， 如Connection: cookie 就是告诉代理服务器，在转发这个请求给原服务器的时候去除cookie
