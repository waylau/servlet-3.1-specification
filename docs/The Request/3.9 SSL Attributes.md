SSL 属性
====

如果请求已经是通过一个安全协议发送，如 HTTPS，必须通过ServletRequest 接口的 isSecure 方法公开该信息。Web 容器必须公开下列属性给 servlet 程序员：

TABLE 3-3 Protocol Attributes

属性 | 属性名称 | Java类型
--- | ----| ----
密码套件 | javax.servlet.request.cipher_suite | String
算法的位大小 | javax.servlet.request.key_size | Integer
SSL 会话 id | javax.servlet.request.ssl_session_id  | String

如果有一个与请求相关的 SSL 证书，它必须由 servlet 容器以java.security.cert.X509Certificate 类型的对象数组暴露给servlet 程序员并可通过一个javax.servlet.request.X509Certificate 类型的 ServletRequest属性访问。

这个数组的顺序是按照信任的升序顺序。证书链中的第一个证书是由客户端设置的，第二个是用来验证第一个的，等等。
