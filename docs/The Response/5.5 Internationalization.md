国际化
====

Servlet 应设置响应的 locale 和字符集。使用ServletResponse.setLocale 方法设置 locale。该方法可以重复的调用；但响应被提交后调用该方法不会产生任何作用。如果在页面被提交之前 servlet 没有设置 locale，容器的默认 locale 将用来确定响应的locale，但是没有制定与客户端通信的规范，例如使用 HTTP 情况下的Content-Language 头。

    <locale-encoding-mapping-list>
        <locale-encoding-mapping>
            <locale>ja</locale>
            <encoding>Shift_JIS</encoding>
        </locale-encoding-mapping>
    </locale-encoding-mapping-list>

如果该元素不存在或没有提供映射，setLocale 使用容器依赖的映射。setCharacterEncoding，setContentType 和 setLocale 方法可以被重复的调用来改变字符编码。如果在 servlet 响应的 getWriter 方法已经调用之后或响应被提交之后，调用相关方法设置字符编码将没有任何作用。只有当给定的上下文类型字符串提供了一个 charset 属性值，调用 setContentType 可以设置字符编码。只有当既没有调用 setCharacterEncoding 也没有调用 setContentType 去设置字符编码之前调用 setLocale 才可以设置字符编码。

在 ServletResponse 接口的 getWriter 方法被调用或响应被提交之前，如果 servlet 没有指定字符编码，默认使用 ISO-8859-1。

如果使用的协议提供了一种这样做的方式，容器必须传递 servlet 响应的writer 使用的 locale 和字符编码到客户端。使用 HTTP 的情况下，locale 可以使用 Content-Language 头传递，字符编码可以作为用于指定文本媒体类型的 Content-Type 头的一部分传递。注意，如果没有指定上下文类型，字符编码不能通过 HTTP 头传递；但是仍使用它来编码通过servlet 响应的 writer 写的文本。
