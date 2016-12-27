# 编程式安全

当仅仅声明式安全是不足以表达应用的安全模型时，编程式安全被用于意识到安全的应用。

编程式安全包括以下 HttpServletRequest 接口的方法：

* authenticate
* login
* logout
* getRemoteUser
* isUserInRole
* getUserPrincipal

login 方法允许应用执行用户名和密码收集（作为一种 Form-Based Login 的替代）。

authenticate 方法允许应用由容器发起在一个不受约束的请求上下文内的来访者请求认证。

logout 方法提供用于应用重置来访者的请求身份。

getRemoteUser 方法由容器返回与该请求相关的远程用户（即来访者）的名字。

isUserInRole 方法确定是否与该请求相关的远程用户（即来访者）在一个特定的安全角色中。

getUserPrincipal 方法确定远程用户（即来访者）的 Principal 名称并返回一个与远程用户相关的 java.security.Principal 对象。调用getUserPrincipal 返回的 Principal 的 getName 方法返回远程用户的名字。这些 API 允许 Servlet 基于获得的信息做一些业务逻辑决策。

如果没有用户通过身份认证，getRemoteUser 方法返回 null，isUserInRole 方法总返回 false，getUserPrincipal 方法总返回null。

isUserInRolem 方法需要一个引用应用角色的参数。对于用在调用isUserInRole 的每一个单独的角色引用，一个带有关联到角色引用的role-name 的 security-role-ref 元素应该声明在部署描述符中。每一个 security-role-ref 元素应该包含一个 role-link 子元素，其值是应用内嵌的角色引用链接到的应用安全角色名称。容器使用 security-role-ref 的 role-name 是否等于角色引用来决定哪一个 security-role 用于测试用户是否在身份中。

例如，映射安全角色应用“FOO”到 role-name 为"manager"的安全角色的语法是：

    <security-role-ref>
        <role-name>FOO</role-name>
        <role-link>manager</role-link>
    </security-role-ref>

在这种情况下，如果属于“manager”安全角色的用户调用了 servlet，则调用 isUserInRole("FOO") API的结果是true。

如果用于调用 isUserInRole 的一个角色引用，没有匹配的 security-role-ref 存在 ，容器必须默认以 security-role 的 role-name 等于用于调用的角色引用来测试用户身份。

角色名`*`应该从不用作调用 isUserInRole 的参数。任何以`*`调用isUserInRole 必须返回 false。如果 security-role 的 role-name 使用`**`测试，且应用没有声明一个role-name为`**`的应用 security-role，isUserInRole 必须仅返回 true。

如果用户已经认证；即，仅当 getRemoteUser 和 getUserPrincipal 将同时返回非 null 值。否则，容器必须检查用户身份是否在应用角色中。

security-role-ref 元素声明通知部署人员应用使用的角色引用和必须定义哪一个映射。
