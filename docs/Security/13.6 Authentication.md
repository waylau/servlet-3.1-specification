# 认证

web客户端可以使用以下机制之一向web服务器认证用户身份：

* HTTP Basic Authentication（HTTP基本认证）
* HTTP Digest Authentication（HTTP摘要认证）
* HTTPS Client Authentication（HTTPS客户端认证）
* Form Based Authentication（基于表单的认证）

### HTTP Basic Authentication

HTTP Basic Authentication 基于用户名和密码，是 HTTP/1.0 规范中定义的认证机制。Web 服务器要求 web 客户端认证用户。作为请求的一部分，web 服务器传递 realm（字符串）给要被认证的用户。Web 客户端获取用户的用户名和密码并传给 web 服务器。Web 服务器然后在指定的realm 认证用户。

基本认证是不安全的认证协议。用户密码以简单的 base64 编码发送，且未认证目标服务器。额外的保护可以减少一些担忧：安全传输机制（HTTPS），或者网络层安全（如 IPSEC 协议或 VPN 策略）被应用到一些部署场景。

### HTTP Digest Authentication

跟 HTTP Basic Authentication 类似，HTTP Digest Authentication 也是 基于用户名和密码，所不同的是，HTTP Digest Authentication 并不在网络中传递用户密码。在 HTTP Digest Authentication 中，客户端发送单向散列的密码（和额外的数据）。尽管密码不在线路上发生，HTTP Digest Authentication 需要对认证容器可用的明文密码等价物（密码等价物可以是这样的，它们仅能在一个特定的 realm 用来认证用户），以致容器可以通过计算预期的摘要验证接收到的认证者。Servlet容器应支持 HTTP_DIGEST 身份认证。

### Form Based Authentication

“登录界面”的外观在使用 web 浏览器的内置的认证机制时不能被改变。本规范引入了所需的基于表单的认证机制，允许开发人员控制登录界面的外观。

Web 应用部署描述符包含登录表单和错误页面条目。登录界面必须包含用于输入用户名和密码的字段。这些字段必须分别命名为 j_username 和j_password。

当用户试图访问一个受保护的 web 资源，容器坚持用户的认证。如果用户已经通过认证则具有访问资源的权限，请求的 web 资源被激活并返回一个引用。如果用户未被认证，发生所有如下步骤：

1. 与安全约束关联的登录界面被发送到客户端，且 URL 路径和 HTTP 协议方法触发容器存储的认证。
2. 用户被要求填写表单，包括用户名和密码字段。
3. 客户端 post 表单到服务器。
4. 容器尝试使用来自表单的信息认证用户。
5. 如果认证失败，使用 forward 或 redirect 返回错误页面，且响应状态码设置为200。错误页面包含失败信息。
6. 如果授权成功，客户端使用存储的 URL 路径重定向到资源。
7. 当一个重定向的和已认证的请求到达容器，容器恢复请求和 HTTP 协 议方法，且已认证的用户主体被检查看看是否它在已授权的允许访问资源的角色中。
8. 如果用户已授权，容器处理接受的请求。

到达步骤7的重定向的请求的 HTTP 协议方法，可以和触发认证的请求有不同的 HTTP 方法。同样地，在第6步的重定向之后，表单认证器必须处理重定向的请求，即使对到达请求的 HTTP 方法的认证不是必需的。为了改善重定向的请求的 HTTP 方法的可预测性，容器应该使用303状态码（SC_SEE_OTHER）重定向（步骤6），除了与HTTP 1.0用户代理的协作之外的是必需的；在这种情况应该使用302状态码。

当进行一个不受保护的传输时，基于表单的认证受制于一些与基本验证一样的相同的脆弱性。

当触发认证的请求在一个安全传输之上到达，或者登录页面受制于一个CONFIDENTIAL user-data-constraint，登录页面必须返回给用户，并在安全传输之上提交到容器。

登录页面受制于一个CONFIDENTIAL user-data-constraint，且一个CONFIDENTIAL user-data-constraint应该包含在每一个包含认证要求的security-constraint中。

HttpServletRequest 接口的 login 方法提供另一种用于应用控制它的登录界面外观的手段。

#### 登录表单

基于表单的登录和基于 URL 的 session 跟踪可以通过编程实现。基于表单的登录应该仅被用在当 session 由 cookie 或 SSL session 信息维护时。

为了进行适当的认证，登录表单的 action 总是 j_security_check。该限制使得不管请求什么资源，登录表单都能工作，且避免了要求服务器指定输出表单的 action 字段。登录表单应该在密码表单字段上指定autocomplete="off"。

下面的示例展示了如何把表单编码到HTML页中：

    <form method=”POST” action=”j_security_check”>
    <input type=”text” name=”j_username”>
    <input type=”password” name=”j_password” autocomplete=”off”>
    </form>
  
如果因为 HTTP 请求造成基于表单的登录被调用，容器必须保存原始请求参数，在成功认证时使用，它重定向调用所请求的资源。

如果用户已使用表单登录通过认证，且已经创建一个 HTTP session，该session 的超时或失效将导致用户被注销，在这种情况下，随后的请求必须导致用户重新认证。注销与认证具有相同的作用域：例如，如果容器支持单点登录，如 Java EE 技术兼容的web容器，用户只需要与托管在web容器中的任何一个 web 应用重新认证即可。

#### HTTPS Client Authentication

使用HTTPS（HTTP over SSL）认证最终用户是一种强认证机制。该机制需要客户端拥有 Public Key Certificate（PKC）。目前，PKC 在电子商务应用中是很有用的，也对浏览器中的单点登录很有用。
 
### 其他容器认证机制

Servlet 容器应该提供公共接口，可用于集成和配置其他的 HTTP 消息层的认证机制，提供给代表已部署应用的容器使用。这些接口应该提供给参与者使用而不是容器供应商（包括应用开发人员、系统管理人员和系统集成人员）。

为了便于实现和集成其他容器认证机制，建议为所有 Servlet 容器实现Servlet 容器 Profile 的 Java 认证 SPI（即，JSR 196）。SPI可下载地址：http://www.jcp.org/en/jsr/detail?id=196
