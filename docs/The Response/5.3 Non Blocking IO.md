非阻塞 IO
====

非阻塞 IO 仅对在 Servlet 和 Filter（2.3.3.3节定义的，“异步处理”）中的异步请求处理和升级处理（2.3.3.5节定义的，“升级处理”）有效。否则，当调用 ServletInputStream.setReadListener 或ServletOutputStream.setWriteListener 方法时将抛出IllegalStateException。为了支持在 Servlet 容器中的非阻塞写，除了在3.7节描述的“非阻塞IO” 对 ServletRequest 做的更改之外，下面做出的更改以便于处理响应相关的类/接口。

WriteListener 提供了如下适用于容器调用的回调方法。

* WriteListener
  * void onWritePossible(). 当一个 WriteListener 注册到ServletOutputStream 时，当可以写数据时该方法将被容器首次调用。当且仅当下边描述的 ServletOutputStream的isReady 方法返回false，容器随后将调用该方法。
  * onError(Throwable t). 当处理响应过程中出现错误时回调。
除了 WriteListener 外，还有如下方法被添加到 ServletOutputStream类并允许开发人员运行时检查是否可以
* ServletOutputStream
  * boolean isReady(). 如果往 ServletOutputStream 写会成功，则该方法返回 true，其他情况会返回 false。如果该方法返回 true，可以在 ServletOutputStream 上执行写操作。如果没有后续的数据能写到 ServletOutputStream，那么直到底层的数据被刷出之前该方法将一直返回 false。且在此时容器将调用 WriteListener 的onWritePossible 方法。随后调用该方法将返回 true。
  * void setWriteListener(WriteListener listener). 关联WriteListener 和当前的 ServletOutputStream，当ServletOutputStream 可以写入数据时容器会调用WriteListener的回调方法。注册了 WriteListener 将开始非阻塞IO。此时再切换到传统的阻塞 IO 是非法的。
  
容器必须线程安全的访问 WriteListener 中的方法。
