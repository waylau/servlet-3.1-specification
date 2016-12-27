什么是过滤器
====

过滤器是一种代码重用的技术，它可以转换 HTTP 请求的内容，响应，及头信息。过滤器通常不产生响应或像 servlet 那样对请求作出响应，而是修改或调整到资源的请求，修改或调整来自资源的响应。

过滤器可以作用于动态或静态内容。这章说的动态和静态内容指的是 Web 资源。

供开发人员使用的过滤器功能有如下几种类型：

* 在执行请求之前访问资源。
* 在执行请求之前处理资源的请求。
* 用请求对象的自定义版本包装请求对请求的header和数据进行修改。
* 用响应对象的自定义版本包装响应对响应的header和数据进行修改。
* 拦截资源调用之后的调用。
* 作用在一个Servlet，一组Servlet，或静态内容上的零个，一个或多个拦截器按指定的顺序执行

### 过滤器组件示例

* Authentication filters  //用户身份验证过滤器
* Logging and auditing filters   //日志记录与审计过滤器
* Image conversion filters  //图片转换过滤器
* Data compression filters  //数据压缩过滤器
* Encryption filters  //加密过滤器
* Tokenizing filters  //分词过滤
* Filters that trigger resource access events  //触发资源访问事件过滤
* XSL/T filters that transform XML content
* MIME-type chain filters  //MIME-TYPE 链过滤器
* Caching filters //缓存过滤器
