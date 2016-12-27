# 编程式安全策略配置

本章定义的注解和API提供用于配置 Servlet 容器强制的安全约束。

### @ServletSecurity 注解

@ServletSecurity 提供了用于定义访问控制约束的另一种机制，相当于那些通过在便携式部署描述符中声明式或通过 ServletRegistration 接口的 setServletSecurity 方法编程式表示。Servlet 容器必须支持在实现 javax.servlet.Servlet 接口的类（和它的子类）上使用@ServletSecurity 注解。

    package javax.servlet.annotation;

    @Inherited
    @Documented
    @Target(value=TYPE)
    @Retention(value=RUNTIME)
    public @interface ServletSecurity {
        HttpConstraint value();
        HttpMethodConstraint[] httpMethodConstraints();
    }

TABLE 13-1 The ServletSecurity Interface

元素 | 描述 | 默认
--- | ---- | ----
value | HttpConstraint 定义了应用到没有在httpMethodConstraints 返回的数组中表示的所有HTTP方法的保护。 | @HttpConstraint
httpMethodConstraints | HTTP方法的特定限制数组 | {}

#### @HttpConstraint

@HttpConstraint 注解用在 @ServletSecurity 中表示应用到所有 HTTP 协议方法的安全约束，且 HTTP 协议方法对应的@HttpMethodConstraint 没有出现在 @ServletSecurity 注解中。

对于一个 @HttpConstraint 返回所有默认值发生在与至少一个@HttpMethodConstraint 返回不同于所有默认值的组合的特殊情况，@HttpMethodConstraint 表示没有安全约束被应用到任何 HTTP 协议方法，否则一个安全约束将应用。这个例外是确保这些潜在的非特定@HttpConstraint 使用没有产生约束，这将明确建立不受保护的访问这些方法；因为，它们没有被约束覆盖。

    package javax.servlet.annotation;

    @Documented
    @Retention(value=RUNTIME)
    public @interface HttpConstraint {
        ServletSecurity.EmptyRoleSemantic value();
        java.lang.String[] rolesAllowed();
        ServletSecurity.TransportGuarantee transportGuarantee();
    }

元素 | 描述 | 默认
--- | ---- | ----
value | 当rolesAllowed返回一个空数组，（只）应用的默认授权语义。 | PERMIT
rolesAllowed | 包含授权角色的数组 | {}
transportGuarantee | 在连接的请求到达时必须满足的数据保护需求。 | NONE

### @HttpMethodConstraint

@HttpMethodConstraint 注解用在 @ServletSecurity 注解中表示在特定 HTTP 协议消息上的安全约束。

    package javax.servlet.annotation;

    @Documented
    @Retention(value=RUNTIME)
    public @interface HttpMethodConstraint {
        ServletSecurity.EmptyRoleSemantic value();
        java.lang.String[] rolesAllowed();
        ServletSecurity.TransportGuarantee transportGuarantee();
    }

TABLE 13-3 The HttpMethodConstraint Interface

元素 | 描述 | 默认
--- | ---- | ----
value | HTTP协议方法名 |
emptyRoleSemantic | 当rolesAllowed返回一个空数组，（只）应用的默认授权语义。| PERMIT
rolesAllowed | 包含授权角色的数组 | {}
transportGuarantee | 在连接的请求到达时必须满足的数据保护需求。 | NONE

@ServletSecurity 注解可以指定在(更准确地说，目标是) Servlet 实现类上，且根据 @Inherited 元注解定义的规则，它的值是被子类继承的。至多只有一个 @ServletSecurity 注解实例可以出现在 Servlet 实现类上，且 @ServletSecurity 注解必须不指定在(更准确地说，目标是) Java 方法上。

当一个或多个 @HttpMethodConstraint 注解定义在 @ServletSecurity注解中时，每一个 @HttpMethodConstraint 定义的 security-constraint，其应用到 @HttpMethodConstraint 中标识的 HTTP 协议方法。除了它的 @HttpConstraint 返回所有默认值、和它包含至少一个返回不同于所有默认值的 @HttpMethodConstraint 的情况之外，@ServletSecurity 注解定义另一个 security-constraint 应该到所有还没有定义相关 @HttpMethodConstraint 的 HTTP 协议方法。

定义在便携式部署描述符中的 security-constraint 元素用于对所有出现在该约束中的 url-pattern 授权。

当在便携式部署描述符中的一个 security-constraint 包含一个 url-pattern，其精确匹配一个使用@ServletSecurity注解的模式映射到的类，该注解必须在由容器在该模式上强制实施的约束上没有效果。

当为便携式部署描述符定义了 metadata-complete=true 时，@ServletSecurity 注解不会应用到部署描述符中的任何 url-pattern 映射到（任何servlet映射到）的注解类。

@ServletSecurity 注解不应用到 ServletRegistration 使用ServletContext 接口的 addServlet(String, Servlet) 方法创建的url-pattern，除非该 Servlet 是由 ServletContext 接口的createServlet 方法构建的。
除了上面列出的，当一个 Servlet 类注解了 @ServletSecurity，该注解定义的安全约束应用到所有 url-pattern 映射到的所有 Servlet 映射到的类。

当一个类没有加 @ServletSecurity 注解时，应用到从那个类映射到的Servlet 的访问策略是由合适的 security-constraint 元素确定的，如果有，在相关的便携式部署描述符，或者由约束禁止任何这样的标签，则如果有，为目标 servlet 通过 ServletRegistration 接口的setServletSecurity 方法编程式确定的。

####　示例

以下示例演示了　ServletSecurity　注解的使用。

CODE EXAMPLE 13-1 for all HTTP methods, no constraints

    @ServletSecurity
    public class Example1 extends HttpServlet {
    }

CODE EXAMPLE 13-2 for all HTTP methods, no auth-constraint, confidential transport　required

    @ServletSecurity(@HttpConstraint(transportGuarantee =
    TransportGuarantee.CONFIDENTIAL))
    public class Example2 extends HttpServlet {
    }

CODE EXAMPLE 13-3 for all HTTP methods, all access denied

    @ServletSecurity(@HttpConstraint(EmptyRoleSemantic.DENY))
    public class Example3 extends HttpServlet {
    }

CODE EXAMPLE 13-4 for all HTTP methods, auth-constraint requiring membership in Role　R1

    @ServletSecurity(@HttpConstraint(rolesAllowed = "R1"))
    public class Example4 extends HttpServlet {
    }

CODE EXAMPLE 13-5 for All HTTP methods except GET and POST, no constraints; forethods GET and POST, auth-constraint requiring membership in　Role R1; for POST, confidential transport required

    @ServletSecurity((httpMethodConstraints = {
    @HttpMethodConstraint(value = "GET", rolesAllowed = "R1"),
    @HttpMethodConstraint(value = "POST", rolesAllowed = "R1",
    transportGuarantee = TransportGuarantee.CONFIDENTIAL)
    })
    public class Example5 extends HttpServlet {
    }

CODE EXAMPLE 13-6 for all HTTP methods except GET auth-constraint requiring　membership in Role R1; for GET, no constraints

    @ServletSecurity(value = @HttpConstraint(rolesAllowed = "R1"),
    httpMethodConstraints = @HttpMethodConstraint("GET"))
    public class Example6 extends HttpServlet {
    }

CODE EXAMPLE 13-7 for all HTTP methods except TRACE, auth-constraint requiring　membership in Role R1; for TRACE, all access denied

    @ServletSecurity(value = @HttpConstraint(rolesAllowed = "R1"),
    httpMethodConstraints = @HttpMethodConstraint(value="TRACE",
    emptyRoleSemantic = EmptyRoleSemantic.DENY))
    public class Example7 extends HttpServlet {
    }

#### 映射 @ServletSecurity 为 security-constraint

本节将介绍 @ServletSecurity 注解映射为它等价表示，security-constraint元素。这提供了使用已存在容器的 security-constraint 实施机制来简化实施。由 Servlet 容器实施的 @ServletSecurity 注解必须在实施的效果上是等价的，由容器从在本节中定义的映射产生security-constraint 元素。

@ServletSecurity 注解用于定义一个方法无关的 @HttpConstraint，且紧跟着一个包含零个或多个 @HttpMethodConstraint 规格的列表。方法无关的约束应用到那些没有定义 HTTP 特定方法约束的所有 HTTP 方法。
当没有包含 @HttpMethodConstraint 元素，@ServletSecurity 注解相当于包含一个 web-resource-collection 的单个 security-constraint 元素，且 web-resource-collection 不包含 http-method 元素，因此涉及到所有 HTTP 方法。

下面的例子展示了把一个不包含 @HttpMethodConstraint 注解的@ServletSecurity 注解表示为单个 security-constraint 元素。相关的 servlet（registration）定义的 url-pattern 元素将被包含在web-resource-collection 中， 任何包含的 auth-constraint 和 user-data-constraint 元素的存在和值，将由定义在13.4.1.3节的“映射 @HttpConstraint 和 @HttpMethodConstraint 为XML”的映射的@HttpConstraint 的值确定。

CODE EXAMPLE 13-8 mapping @ServletSecurity with no contained
@HttpMethodConstraint

    @ServletSecurity(@HttpConstraint(rolesAllowed = "Role1"))

    <security-constraint>
        <web-resource-collection>
            <url-pattern>...</url-pattern>
        </web-resource-collection>
        <auth-constraint>
            <role-name>Role1</role-name>
        </auth-constraint>
    </security-constraint>

当指定了一个或多个 @HttpMethodConstraint 元素，方法无关的约束关联一个单个 security-constraint 元素，其，web-resource-collection 包含了为每一个 HTTP 方法命名在 @HttpMethodConstraint 元素中的 http-method-omission 元素。如果方法无关的约束返回所有默认值和至少一个 @HttpMethodConstraint 不是，包含 http-method-omission 元素的 security-constraint 必须不被创建。每一个 @HttpMethodConstraint 与另一种包含一个 web-resource-collection 的 security-constraint 关联，web-resource-collection 包含一个使用相应 HTTP 方法命名的 http-method 元素。

下面的例子展示了映射带有单个 @HttpMethodConstraint 的@ServletSecurity 注解为两种 security-constraint 元素。相应的Servlet（registration）定义的 url-pattern 元素将被包含在两种约束的 web-resource-collection 中，且任何包含的 auth-constraint 和 user-data-constraint 元素的存在和值，将由定义在13.4.1.3节的“映射 @HttpConstraint 和 @HttpMethodConstraint 为XML”的映射关联的 @HttpConstraint 和 @HttpMethodConstraint 的值确定。

CODE EXAMPLE 13-9 mapping @ServletSecurity with contained @HttpMethodConstraint

    @ServletSecurity(value=@HttpConstraint(rolesAllowed = "Role1"),
    httpMethodConstraints = @HttpMethodConstraint(value = "TRACE",
    emptyRoleSemantic = EmptyRoleSemantic.DENY))

    <security-constraint>
        <web-resource-collection>
            <url-pattern>...</url-pattern>
            <http-method-omission>TRACE</http-method-omission>
        </web-resource-collection>
        <auth-constraint>
            <role-name>Role1</role-name>
        </auth-constraint>
    </security-constraint>
    <security-constraint>
        <web-resource-collection>
            <url-pattern>...</url-pattern>
            <http-method>TRACE</http-method>
        </web-resource-collection>
        <auth-constraint/>
    </security-constraint>

### 映射 @HttpConstraint 和 @HttpMethodConstraint 为 XML

本节将介绍映射映射 @HttpConstraint 和 @HttpMethodConstraint 注解值（在 @ServletSecurity 中定义使用的）为它们等价的 auth-constraint 和 user-data-constraint 表示，这些注解共用一个通用模型用于表示用在便携式部署描述符中的 auth-constraint 和 user-data-constraint 元素的等价形式。该模型包括以下3种元素：

* emptyRoleSemantic
授权语义，PERMIT或DENY，适用于在rolesAllowed中没有指定的角色时。此元素的默认值为PERMIT，且DENY不支持与非空的rolesAllowed列表结合使用。
* rolesAllowed
一个包含授权角色的名字列表。当该列表为空时，其含义取决于emptyRoleSemantic的值。当角色名字“*”包含在允许的角色列表中时是没有特别的含义的。当特殊的角色名字“**”出现在rolesAllowed中时，它表示用户认证，不受约束的角色，是必需的和足够的。该元素的默认值是一个空列表。
* transportGuarantee
数据保护需求，NONE 或 CONFIDENTIAL，在连接的请求到达时必须满足。该元素与一个包含一个使用相应值的transport-guarantee的user-data-constraint是等价的。该元素的默认值是NONE。
下面的例子展示了上述的 @HttpConstraint 模型和 web.xml 中的 auth-constraint 和 user-data-constraint 元素之间的对应关系。

CODE EXAMPLE 13-10 emptyRoleSemantic=PERMIT, rolesAllowed={}, transportGuarantee=NONE

没有 constraint

CODE EXAMPLE 13-11 emptyRoleSemantic=PERMIT, rolesAllowed={}, transportGuarantee=CONFIDENTIAL

    <user-data-constraint>
        <transport-guarantee>CONFIDENTIAL</transport-guarantee>
    </user-data-constraint>

CODE EXAMPLE 13-12 emptyRoleSemantic=PERMIT, rolesAllowed={Role1},transportGuarantee=NONE

    <auth-constraint>
        <security-role-name>Role1</security-role-name>
    </auth-constraint>

CODE EXAMPLE 13-13 emptyRoleSemantic=PERMIT, rolesAllowed={Role1},transportGuarantee=CONFIDENTIAL

    <auth-constraint>
        <security-role-name>Role1</security-role-name>
    </auth-constraint>
    <user-data-constraint>
        <transport-guarantee>CONFIDENTIAL</transport-guarantee>
    </user-data-constraint>

CODE EXAMPLE 13-14 emptyRoleSemantic=DENY, rolesAllowed={}, transportGuarantee=NONE

    <auth-constraint/>

CODE EXAMPLE 13-15 emptyRoleSemantic=DENY, rolesAllowed={}, transportGuarantee=CONFIDENTIAL

    <auth-constraint/>
    <user-data-constraint>
        <transport-guarantee>CONFIDENTIAL</transport-guarantee>
    </user-data-constraint>

### ServletRegistration.Dynamic 的 setServletSecurity

ServletContextListener 内的 setServletSecurity 方法用于定义应用到 ServletRegistration 定义的映射的安全约束。

    Collection<String> setServletSecurity(ServletSecurityElement arg);

setServletSecurity 的 javax.servlet.ServletSecurityElement 参数与 ServletSecurity 接口的 @ServletSecurity 注解在结构和模型上是类似的。因此，定义在13.4.1.2节的“映射@ServletSecurity为security-constraint”的映射，应用类似的包含HttpConstraintElement 和 HttpMethodConstraintElement 值的ServletSecurityElement 映射为其等价的 security-constraint 表示。

setServletSecurity 方法返回一组 URL pattern（可能空），其已是便携式部署描述符中的 security-constraint 元素的精确目标（因此，调用是不影响的）。

如果 ServletContext 中得到的 ServletRegistration 已经被初始化了，该方法抛出 IllegalStateException。

当便携式部署描述符中的 security-constraint 包含一个 url-pattern 其精确匹配 ServletRegistration 映射的 pattern，调用ServletRegistration 的 setServletSecurity 必须对 Servlet 容器对 pattern 实施的约束没有任何影响。

除了上面列出的，包括当 Servlet 类注解了 @ServletSecurity，当调用了 ServletRegistration 的 setServletSecurity，它制定应用到registration 的 url-pattern 的安全约束。
