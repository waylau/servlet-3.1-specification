# 映射规范

在web应用部署描述符中，以下语法用于定义映射：

* 以‘/’字符开始、以‘/*’后缀结尾的字符串用于路径匹配。
* 以‘*.’开始的字符串用于扩展名映射。
* 空字符串“”是一个特殊的URL模式，其精确映射到应用的上下文根，即，http://host:port/<context-root>/请求形式。在这种情况下，路径信息是‘/’且servlet路径和上下文路径是空字符串（“”）。
* 只包含“/”字符的字符串表示应用的“default”servlet。在这种情况下，servlet路径是请求URL减去上下文路径且路径信息是null。
* 所有其他字符串仅用于精确匹配。

如果一个有效的 web.xml（在从 fragment 和注解合并了信息后）中有任意一个url-pattern，其映射到多个 servlet，那么部署将失败。

### 隐式映射

如果容器有一个内部的JSP容器，*.jsp扩展名映射到它，允许执行JSP页面的要求。该映射被称为隐式映射。如果Web应用定义了一个*.jsp映射，它的优先级高于隐式映射。

Servlet 容器允许进行其他的隐式映射，只要显示映射的优先。例如，一个 *.shtml 隐式映射可以映射到包含在服务器上的功能。

### 示例映射集合

请看下面的一组映射：

TABLE 12-1 Example Set of Maps

路径模式 | Servlet
---- | ----
/foo/bar/* | servlet1
/baz/* | servlet2
/catalog | servlet3
*.bop | servlet4

将产生以下行为：

TABLE 12-2 Incoming Paths Applied to Example Maps

访问的路径  | Servlet 处理请求
---- | ----
/foo/bar/index.html | servlet1
/foo/bar/index.bop | servlet1
/baz | servlet2
/baz/index.html | servlet2
/catalog | servlet3
/catalog/index.html | “default” servlet
/catalog/racecar.bop | servlet4
/index.bop | servlet4

请注意，在 /catalog/index.html 和 /catalog/racecar.bop 的情况下，不使用映射到“/catalog”的 servlet，因为不是精确匹配的。
