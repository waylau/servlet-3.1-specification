# 错误处理

### 请求属性

在发生错误时，Web 应用程序必须能够详细说明，应用程序中的其他资源被用来提供错误响应的内容主体。这些资源的规定在部署描述文件中配置。

如果错误处理位于一个servlet或JSP页面：

* 原来打开的由容器创建的请求和响应对象被传递给servlet或JSP页面。。
* 请求路径和属性被设置成如同RequestDispatcher.forward跳转到已经完成的错误资源一样。
* 必须设置表10-1中的请求属性。

TABLE 10-1 Request Attributes and their types

请求属性  | 类型
------- | ------
javax.servlet.error.status_code  | java.lang.Integer
javax.servlet.error.exception_type | java.lang.Class
javax.servlet.error.message | java.lang.String
javax.servlet.error.exception | java.lang.Throwable
javax.servlet.error.request_uri | java.lang.String
javax.servlet.error.servlet_name | java.lang.String

这些属性允许 servlet 根据状态码、异常类型、错误消息、传播的异常对象、发生错误时由 servlet 处理的请求 URI（像调用 getRequestURI方法确定的 URI 一样）、以及发生错误的 servlet 的逻辑名称来生成专门的内容。

由于本规范的2.3版本引入了异常对象属性列表，异常类型和错误消息属性是多余的。他们保留向后兼容早期的 API 版本。

### 错误页面

为了使开发人员能够在 servlet 产生一个错误时自定义内容的外观返回到 Web 客户端，部署描述文件中定义了一组错误页面说明。这种语法允许当 servlet 或过滤器调用响应对象的 sendError 方法指定状态码时，或如果 servlet 产生一个异常或错误传播给容器时，由容器返回资源配置。

如果调用应对象的 sendError 方法，容器参照为 Web 应用声明的错误页面列表，使用状态码语法并试图匹配一个错误页面。如果找到一个匹配的错误页面，容器返回这个位置条目指示的资源。

在处理请求的时候 servlet 或过滤器可能会抛出以下异常：

* 运行时异常或错误
* ServletException或它的子类异常
* IOException或它的子类异常

Web 应用程序可以使用 exception-type 元素声明错误页面。在这种情况下，容器通过比较抛出的异常与使用 exception-type 元素定义的error-page 列表来匹配异常类型。在容器中的匹配结果返回这个位置条目指示的资源。在类层次中最接近的匹配将被返回。

如果声明的 error-page 中没有包含 exception-type 适合使用的类层次结构的匹配，那么抛出一个 ServletException 异常或它的子类异常，容器通过 ServletException.getRootCause 方法提取包装的异常。第二遍通过修改错误页面声明，使用包装的异常再次尝试匹配声明的错误页面。

使用 exception-type 元素声明的 error-page 在部署描述文件中必须唯一的，由 exception-type 的类名决定它的唯一性。同样地， 使用status-code 元素声明的 error-page 在部署描述文件中必须是唯一的，由状态码决定它的唯一性。

如果部署描述中的一个 error-page 元素没包含一个 exception-type 或  error-code 元素，错误页面是默认的错误页面。

当错误发生时，错误页面机制不会干预调用使用 RequestDispatcher 或filter.doFilter 方法。用这种方法，过滤器或 Servlet 有机会使用RequestDispatcher 处理产生的错误。

如果上述错误页面机制没有处理 servlet 产生的错误，那么容器必须确保发送一个状态500的响应。

默认的 servlet 和容器将使用 sendError 方法，发送4xx和5xx状态的响应，这样错误机制才可能会被调用。默认的servlet和容器将使用setStatus 方法，设置2xx和3xx的响应，并不会调用错误页面机制。

如果应用程序使用第2.3.3.3节，“异步处理”中描述的异步操作，那么处理应用程序创建的线程的所有错误是应用程序的职责。容器应该通过AsyncContext.start 方法注意线程发出的错误。对于处理AsyncContext.dispatch 过程中发生的错误，请参照相关章节，“执行dispatch 方法的时候可能发生的错误或异常必须被容器按照如下的方式捕获并处理”。

### 错误过滤器

错误页面机制运行在由容器创建的原来未包装过的或未经过过滤的请求或响应对象上。在第6.2.5节“过滤器和请求转发”中描述的机制可以在产生一个错误响应之前用来指定要应用的过滤器。
