# 从我们把 www.baidu.com 丢进浏览器地址栏，到显示出百度首页，中间发生了什么？
1. 浏览器会辨识访问地址所需要的协议，直接把 **www.baidu.com** 放入地址栏，浏览器会使用默认协议 HTTP 尝试访问，而 **百度** 从 2015 年完成全站 HTTPS 访问改造，所以浏览器最终访问的完整地址为： **https://www.baidu.com/**。 HTTPS 与 HTTP 协议内容基本相同，除了，通过 HTTPS 访问时浏览器和服务器之间会建立一个加密机制，双方通信内容仅彼此可见；此外，浏览器还可以通过请求服务器的证书，来验证服务器的可靠性。
2. 浏览器生成请求报文，HTTP 协议为文本协议，报文直接可见（简略）：
```
Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding:gzip, deflate, br
Accept-Language:zh-CN,zh;q=0.9,en;q=0.8,zh-TW;q=0.7,ja;q=0.6,fr;q=0.5
Cache-Control:no-cache
Connection:keep-alive
Host:www.baidu.com
Pragma:no-cache
Upgrade-Insecure-Requests:1
User-Agent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.94 Safari/537.36
```
3. 虽然浏览器和服务器之间通信协议为 HTTP，但是在通信中负责传输数据的还是 TCP/IP 协议。浏览器生成请求报文之后，向操作系统发起访问地址： **www.baidu.com** 的请求（或者说「系统调用」）。操作系统依据 IP 协议内容，查找出百度服务器的实际 IP 地址。然后依据 TCP 协议与服务器建立可信的通信通道，并且将包装好的 HTTP 报文分割成多个报文段，按照序号将每个报文段可靠的传输给百度的服务器。
4. 收到浏览器请求的百度服务器开始处理请求，像百度这样的大型网站还会有一个前置服务器以水平扩展服务的响应能力。前置服务器仅做分发用途，收到请求立刻转给对应的后置服务器，后置服务器处理完成之后，向浏览器发送响应报文。
5. 响应报文分为：「响应头部」和「响应内容」两个部分。「响应头部」与「请求头部」报文格式类似（简略）：
```
HTTP/1.1 200 OK 
Cache-Control: private
Connection: Keep-Alive
Content-Encoding: gzip
Content-Type: text/html; charset=utf-8
Date: Mon, 11 Dec 2017 05:10:01 GMT
Expires: Mon, 11 Dec 2017 05:09:34 GMT
Server: BWS/1.1
Strict-Transport-Security: max-age=172800
Vary: Accept-Encoding
X-Powered-By: HPHP
X-Ua-Compatible: IE=Edge,chrome=1
Transfer-Encoding: chunked
```
而对于百度首页来说，「响应内容」就是首页的 HTML 代码。
6. 与请求的传输过程类似，服务器的响应报文，也是通过 TCP/IP 协议进行传输。
7. 收到响应的浏览器开始可以分为两部分的工作：「解析」与「渲染」。
8. 解析可以分为三种：
    1. 解析 HTML （或者 XHTML 与 SVG），生成 DOM Tree；
    2. CSS，解析 CSS 生成 CSS Rule Tree；
    3. JavaScript，通过 DOM API 与 CSSOM API 操作 DOM Tree 与 CSS Rule Tree。
9. 解析完成后，浏览器引擎会通过DOM Tree 和 CSS Rule Tree 来构造 Rendering Tree。
10. 在解析 HTML 的过程中，如遇到外部资源的引用时，会加载外部资源（如：图片、字体文件，外联的 CSS 、JavaScript 文件）。外部 CSS 与 JavaScript 文件会阻塞浏览器渲染过程，而其他外部资源（如图片、字体）不阻塞渲染。外部资源的请求/响应过程，与 1 至 5 步类似。
11. 解析与渲染是会重复的过程。当 DOM Tree 上的元素发生变化时，如果元素的尺寸不变，颜色发生变化，会产生 Repaint 操作； 如果元素的尺寸也发生了变化，这时会重新验证并计算 Rendering Tree，产生 Reflow 操作。
12. 最后再通过调用操作系统的 GUI API 进行绘制工作。
![百度](http://upload-images.jianshu.io/upload_images/53309-2c622450979b215a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
