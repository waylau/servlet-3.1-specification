可插拔性
====

### web.xml模块

使用上述定义的注解，使得 web.xml 的使用变为可选。然而，对于覆盖默认值或使用注解设置的值，仍然需要使用部署描述符。如前所述，如果 web.xml 描述符中的 metadata-complete 元素设置为 true，则存在于 class 文件和绑定在 jar 包中的 web-fragments 中的指定部署信息的注解将不被处理。这意味着，所有应用的元数据通过 web.xml 描述符指定。

为了给开发人员更好的可插拔性和更少的配置，在这个版本的规范中，我们引入了 web 模块部署描述符片段（web fragment）的概念。web fragment 是 web.xml 的部分或全部，可以在一个类库或框架 jar 包的META-INF 目录指定和包括。在 WEB-INF/lib 目录中的普通的老的 jar 文件即使没有 web-fragment.xml 也可能被认为是一个 fragment，任何在它中指定的注解都将按照定义在8.2.3节的规则处理，容器将会取出并按照如下定义的规则进行配置。

web fragment 是 web 应用的一个逻辑分区，以这样一种方式，在应用中使用的框架可以定义所有制品（artifact）而无需要求开发人员在web.xml中 编辑或添加信息。它几乎包含 web.xml 描述符中使用的所有相同元素。不过描述符的顶级元素必须 是web-fragment 且对应的描述符文件必须被称为 web-fragment.xml，相关元素的顺序在 web-fragment.xml 和 web.xml 也是不同的，请参考定义在第14章的部署描述符一章中对应的 web-fragment schema。

如果框架打包成 jar 文件，且有部署描述符的形式的元数据信息，那么web-fragment.xml 描述符必须在该 jar 包的 META-INF/ 目录中。
如果框架想使用 META-INF/web-fragment.xml，以这样一种方式，它扩充了 web 应用的 web.xml，框架必须被绑定到 Web应用的 WEB-INF/lib目录中。为了使框架中的任何其他类型的资源（例如 ，类文件）对web应用可用，把框架放置在 web 应用的 classloader 委托链的任意位置即可。换句话说，只有绑定到 web 应用的 WEB-INF/lib 目录中的 JAR 文件，但不是那些在类装载委托链中更高的，需要扫描其 web-fragment.xml。

在部署期间，容器负责扫描上面指定的位置和发现 web-fragment.xml 并处理它们。存在于当前的单个 web.xml 的名字唯一性的要求，也同样适用于一组 web.xml 和所有能适用的 web-fragment.xml 文件。

如下是库或框架可以包括什么的例子。

    <web-fragment>
        <servlet>
            <servlet-name>welcome</servlet-name>
            <servlet-class>
                WelcomeServlet
            </servlet-class>
        </servlet>
        <listener>
            <listener-class>
                RequestListener
            </listener-class>
        </listener>
    </web-fragment>

以上的 web-fragment.xml 将被包括在框架的 jar 文件的 META-INF/ 目录。web-fragment.xml 配置和应该应用的注解的顺序是未定义的。如果顺序对于某一应用是很重要的方面，请参考下面如何实现所需的顺序定义的规则。

### web.xml 和 web-fragment.xml 顺序

由于规范允许应用配置由多个配置文件组成（web.xml 和 web-fragment.xml）的资源，从应用中多个不同位置发现和加载，顺序问题必须被解决。本节详述了配置资源的作者如何声明他们制品（artifact）的顺序要求。
web-fragment.xml可以有一个javaee:java-identifierType类型的顶级<name>元素，且在一个web-fragment.xml中仅能有一个<name>元素。如果存在一个<name>元素，它必须考虑用于artifact顺序（除非出现重复名异常，如上文所述）。
两种情况必须被考虑，以允许应用程序配置资源来表达它们的顺序配置。
1. 绝对顺序：在 web.xml 中的`<absolute-ordering>`元素。在一个 web.xml 中仅能有一个`<absolute-ordering>`元素。
  * 在这种情况下，第二种情况处理的顺序配置必须被忽略。
  * web.xml 和 WEB-INF/classes 必须在列在 absolute-ordering 元素中的所有 web-fragment 之前处理。
  * `<absolute-ordering>`的任何直接子`<name>`元素必须被解释为在这些被指定的 web-fragment 中表示绝对顺序，不管它存不存在，必须被处理。
  * `<absolute-ordering>`可以包含零个或一个`<others />`元素。下面描述此元素必需的功能。如果`<absolute-ordering>`元素没有包含`<others />`元素，没有在`<name />`明确提到的 web-fragment 必须被忽略。不会扫描被排除的 jar 包中的注解 Servlet、Filter 或Listener。然而，如果是被排除的 jar 包中的 servlet、filter 或 listener，其列在web.xml 或非排除的 web-fragment.xml 中，那么它的注解将使用除非另外使用 metadata-complete 排除。
在排除的 jar 包的 TLD 文件中发现的 ServletContextListener 不能使用编程式 API 配置Filter 和 Servlet，任何试图这样做将导致 IllegalStateException。如果发现的ServletContainerInitializer 是从一个被排除的 jar 包中装载的，它将被忽略。无论是否设置了 metadata-complete，通过`<absolute-ordering>`排除的jar包不扫描被任何ServletContainerInitializer 处理的类。
  * 重复名字异常：如果，当遍历`<absolute-ordering>`子元素，遇到多个子元素具有相同`<name>`元素，只需考虑首次出现的。
2. 相对顺序：在web-fragment.xml中的 `<ordering>`元素，一个 web-fragment.xml 只能有一个`<ordering>`元素。
  * web-fragment.xml 可以有一个`<ordering>`元素。如果是这样，该元素必须包含零个或一个`<before>` 元素和零个或一个`<after>`元素。这些元素的含义在下面进行说明。
  * web.xml 和 WEB-INF/classes 必须在列在 ordering 元素中的所有 web-fragment 之前处理。
  * 重复命名异常：如果，当遍历 web-fragments，遇到多个成员具有相同`<name>`元素，应用必须记录包含帮助解决这个问题的提供有用信息的错误消息，且部署必须失败。例如，一种解决该问题的办法是用户使用绝对顺序，在这种情况下相对顺序被忽略。
  * 思考这个简短的但具说明下的例子。3个 web-fragment - MyFragment1、 MyFragment2 和 MyFragment3 作为应用一部分，同时也包括一个 web.xml。

web-fragment.xml

    <web-fragment>
        <name>MyFragment1</name>
        <ordering><after><name>MyFragment2</name></after></ordering>
        ...
    </web-fragment>

web-fragment.xml

    <web-fragment>
        <name>MyFragment2</name>
        ..
    </web-fragment>

web-fragment.xml

    <web-fragment>
        <name>MyFragment3</name>
        <ordering><before><others/></before></ordering>
        ..
    </web-fragment>

web.xml

    <web-app>
        ...
    </web-app>

在该示例中，处理顺序将是：

web.xml

MyFragment3

MyFragment2

MyFragment1

前面的示例说明了一些,但不是全部,以下是全部原则。

* `<before>` 意味着文档必须被安排在指定在嵌套`<name>`元素的 name匹配的文档之前。
* `<after>` 意味着文档必须被安排在指定在嵌套`<name>`元素的 name匹配的文档之后。
* 在`<before>` 或 `<after>`可以包括特殊的`<others/>`元素零次或一次，或直接包括在`<absolute-ordering>`元素中零次或一次。`<others/>`元素必须作如下处理。
  * 如果`<before>`元素包含一个嵌套的`<others/>`，该文档将被移动到有序的文档列表开头。如果有多个文档指定`<before><others/>`，则它们将都在有序的文档列表开头，但该组文档的顺序是未指定的。
  * 如果`<after>`元素包含一个嵌套的`<others/>`，该文档将被移动有序的文档列表末尾。如果有多个文档指定`<after><others/>`，则它们将都在有序的文档列表末尾，但该组文档的顺序是未指定的。
  *在一个的`<before>`或`<after>`元素内，如果存在一个`<others/>` 元素，但在它的父元素内`<name>`元素不是唯一的，父元素内的其他元素必须按照顺序处理。
  * 如果`<others/>`直接出现在`<absolute-ordering>`内，`runtime`必须确保任何 web-fragment 未明确指定在`<absolute-ordering>`部分的以处理的顺序包括在这一点上。
* 如果 web-fragment.xml 文件没有`<ordering>`或 web.xml 没有`<absolute-ordering>`元素，则artifact被假定没有任何顺序依赖。
* 如果runtime发现循环引用，必须记录提供有用信息的消息，应用必须部署失败。此外， 用户采取的一系列动作可能是在 web.xml 中使用绝对顺序。
* 之前的示例可以被扩展以说明当 web.xml 包含顺序部分的情况

web.xml

    <web-app>
        <absolute-ordering>
            <name>MyFragment3</name>
            <name>MyFragment2</name>
        </absolute-ordering>
        ...
    </web-app>

在该示例中，各种元素的顺序将是：

web.xml

MyFragment3

MyFragment2

下面包括了一些额外的示例场景。所有这些适用于相对顺序且不是绝对顺序。

Document A：

    <after>
        <others/>
        <name>
            C
        </name>
    </after>

Document B:

    <before>
        <others/>
    </before>

Document C:

    <after>
        <others/>
    </after>

Document D： 没有指定顺序

Document E： 没有指定顺序

Document F：

    <before>
        <others/>
        <name>
            B
        </name>
    </before>

产生的解析顺序：

web.xml, F, B, D, E, C, A。

Document <no id>:

    <after>
        <others/>
    </after>
    <before>
        <name>
        C
        </name>
    </before>

Document B:

    <before>
        <others/>
    </before>

Document C: 没有指定顺序

Document D:

    <after>
        <others/>
    </after>

Document E:

    <before>
        <others/>
    </before>

Document F：没有指定顺序

产生的解析顺序可能是下列之一：

* B, E, F, <no id>, C, D
* B, E, F, <no id>, D, C
* E, B, F, <no id>, C, D
* E, B, F, <no id>, D, C
* E, B, F, D, <no id>, C
* B, E, F, D, <no id>, C

Document A：

    <after>
        <name>
            B
        </name>
    </after>

Document B：没有指定顺序

Document C：

    <before>
        <others/>
    </before>

Document D: 没有指定顺序

产生的解析顺序： C, B, D, A。解析的顺序也可能是： C, D, B, A 或 C, B, A, D

### 装配 web.xml、web-fragment.xml 描述符和注解

如果对于一个应用 Listener、Servlet 和 Filter 的调用顺序是很重要的，那么必须使用部署描述符。同样，如果有必要，可以使用上面定义的顺序元素。如上所述，当使用注解定义 Listener、Servlet 和 Filter，它们调用的顺序是未指定的。下面是用于装配应用程序的最终部署描述符的一组规则：

1. 如果有关的 Listener、Servlet 和 Filter 的顺序必须指定，那么必须在 web-fragment.xml 或 web.xml中指定。
2. 顺序将依据它们定义在描述符中的顺序，和依赖于 web.xml 中的 absolute-ordering 元素或 web-fragment.xml 中的 ordering 元素，如果存在的话。
  * 匹配请求的过滤器链的顺序是它们在web.xml中声明的顺序。
  * Servlet在请求处理时实例化或在部署时立即实例化。在后一种情况，以它们的load-on-startup 元素指定的顺序实例化。
  * 在之前发布的规范，上下文Listener以随机顺序调用。在Servlet3.0，Listener以它们在web.xml中声明的顺序调用，如下所示：
    * javax.servlet.ServletContextListener实现的contextInitialized方法以声明时顺序调用，contextDestroyed 以相反顺序调用。
    * javax.servlet.ServletRequestListener 实现的requestInitialized 以声明时顺序调用，requestDestroyed 方法以相反顺序调用。
    * javax.servlet.http.HttpSessionListener 实现的sessionCreated方法以声明时顺序调用，sessionDestroyed 方法以相反顺序调用。
    * 当相应的事件触发时，javax.servlet.ServletContextAttributeListener、javax.servlet.ServletRequestAttributeListener 和javax.servlet.HttpSessionAttributeListener 的方法按照它们声明的顺序调用。
  * 如果在web.xml使用enabled元素禁用引入的servlet，那么该servlet对指定的url-pattern不可用。
  * 当在 web.xml、web-fragment.xml 和 注解之间解析发生冲突时 web 应用的 web.xml 具有最高优先级。
  * 如果没有在描述符中指定metadata-complete 或在部署描述符中设置为false，通过组合出现在注解和描述符中的metadata导出有效的metadata。合并的规则具体如下：
    * 在web fragment中的配置设置用于扩充那些已指定在主web.xml的配置设置，使用这种方式就好像它们指定在同一个web.xml。
    * 添加到主web.xml的web fragment中的配置设置的顺序由8.2.2节“web.xml和web-fragment.xml顺序”指定。
    * 当主web.xml的metadata-complete 属性设置为true，被认为是完整的且在部署时不会扫描注解和fragment。如果有absolute-ordering和ordering元素将被忽略。当设置fragment上的为true时，metadata-complete属性仅适用于在特定的jar包中扫描注解。
    * 除非metadata-complete 设置为true，否则web fragment被合并到主web.xml。合并发生在相关fragment的注解处理之后。
    * 当使用web fragment扩充web.xml时以下被认为配置冲突：
      * 多个<init-param>元素使用相同的<param-name>但不同的<param-value>
      * 多个<mime-mapping>元素使用相同的<extension>但不同的<mime-type>
    * 上面的配置冲突被解析为如下：
      * 在主web.xml和web fragment之间的配置冲突被解析为在web.xml的配置具有高优先级。
      * 在两个web fragment之间的配置冲突，冲突的中心元素没有出现在主web.xml，将导致一个错误。必须记录一个有用的消息，且应用必须部署失败。
    * 上面的冲突被解析后，这些额外的规则适用：
      * 可以在多个web-frament中声明任意多次元素并生成到web.xml。比如，<context-param> 元素可以以不同的名字添加。
      * 如果指定在web.xml中的覆盖了指定在web-fragment中的同名的值，则可以声明任意多次元素。
      * 如果是最少出现零次且最多出现一次的元素存在于web fragment，且没有在主web.xml中，则主web.xml继承web fragment的设置。如果元素出现在主web.xml和web fragment，则主web.xml的配置设置具有高优先级。例如，如果在主web.xml和web fragment中都声明了相同的servlet，且声明在web fragment中的servlet指定了<load-on-startup>元素，且没在主web.xml指定，则web fragment的<load-on-startup>元素将被使用并合并到web.xml。
      * 如果是最少出现零次且最多出现一次的元素指定在两个web fragment，且没有出现在主web.xml，则认为是错误的。例如，如果两个web fragment声明了相同的Servlet，但具有不同的<load-on-startup>元素，且相同的Servlet也声明在主web.xml，但没有<load-on-startup>，则必须报告一个错误。
      * <welcome-file>声明是可添加的。
      * 具有相同<servlet-name>的<servlet-mapping>元素可以添加到多个web-fragment。在web.xml中指定的<servlet-mapping>覆盖在web-fragment中指定的同名的<servlet-name>的<servlet-mapping>。
      * 具有相同<filter-name>的<filter-mapping>元素可以添加到多个web-fragment。在web.xml中指定的<filter-mapping>覆盖在web-fragment中指定的同名的<filter-name>的<filter-mapping>。
      * 具有相同<listener-class>的多个<listener>元素被当作一个<listener>声明。
      * 合并产生的web.xml被认为是<distributable>，仅当所有它的web fragment也被标记为<distributable>。
      * web fragment的顶级<icon>和它的孩子元素，<display-name>，和<description>元素被忽略。
      * jsp-property-group是可添加的。当绑定静态资源到jar包的META-INF/resources目录，推荐jsp-config元素使用url-pattern，反对使用extension映射。此外，如果存在一个fragment的JSP资源，则应该在一个与fragment同名的子目录中。这有助于防止一个web-fragment的jsp-property-group受到来自应用的主docroot中的JSP的影响和受到来自一个fragment的META-INF/resources的JSP的影响。
    * 对于所有资源引用元素 (env-entry, ejb-ref, ejb-local-ref, service-ref, resource-ref, resource-env-ref, message-destination-ref, persistence-context-ref and persistence-unit-ref) 如下规则适用：
      * 如果任意资源引用元素出现在web fragment，主web.xml继承web fragment的值。 如果该元素同时出现在主web.xml和web fragment，使用相同的名字，web.xml具有高优先级。所有fragment的子元素除下面指定的injection-target被合并到主web.xml。例如，如果主web.xml和web fragment都使用相同的<resource-ref-name>声明一个<resource-ref>，将使用web.xml中的<resource-ref>且不会合并fragment中的任意子元素除下面声明的<injection-target>。
      * 如果资源引用元素指定在两个fragment，当没有指定在主web.xml中，且资源引用元素的所有属性和子元素都是一样的，资源引用将被合并到主web.xml。如果使用相同名字在两个fragment中指定资源引用元素，且没有在web.xml中指定，属性和子元素是不一样的，那么被认为是错误的。错误必须被报告且应用必须部署失败。例如，如果两个web fragment使用相同的<resource-ref-name>声明了<resource-ref>但类型一个指定为javax.sql.DataSource另一个指定为JavaMail，这是错误的且应用必须部署失败。
      * 对于在fragment中使用相同名称的<injection-target> 的资源引用元素将被合并到主web.xml。
      * 除了上面定义的web-fragment.xml的合并规则之外，下面的规则适用于使用资源引用注解(@Resource, @Resources, @EJB, @EJBs, @WebServiceRef, @WebServiceRefs, @PersistenceContext, @PersistenceContexts,@PersistenceUnit, and @PersistenceUnits)。如果资源引用注解应用到类上，这等价于定义了一个资源，但是这不等价于定义一个injection-target。在这种情况下上述规则适用于injection-target元素。如果在字段上使用资源引用注解，这等价于在web.xml定义injection-target元素。但是如果在描述符中没有injection-target元素，那么fragment中的injection-target仍将被合并到上面定义的web.xml。如果从另一方面来说，在主web.xml中有一个injection-target并同时有一个同资源名的资源引用注解，那么这被认为是对资源引用注解的覆盖。在这种情况下，由于在描述符中指定了一个injection-target，上述定义的规则将适用于除了覆盖的资源引用注解。
    * 如果在两个fragment中指定了data-source元素，而没有出现在主web.xml，且data-source元素的所有属性和子元素都是一样的，data-source将被合并到主web.xml。如果在两个fragment中指定同名的data-source元素，而没有出现在主web.xml且两个fragment的属性和子元素不是一样的，这被认为是错误的。在这种情况下，必须报告一个错误且引用必须部署失败。

下面是一些示例，展示了在不同情况下的结果。

CODE EXAMPLE 8-4

web.xml - 没有 resource-ref 的定义

Fragment 1

web-fragment.xml

    <resource-ref>
        <resource-ref-name="foo">
        ...
        <injection-target>
            <injection-target-class>
                com.foo.Bar.class
            </injection-target-class>
            <injection-target-name>
                baz
            </injection-target-name>
        </injection-target>
    </resource-ref>

有效的metadata将是：

    <resource-ref>
        <resource-ref-name="foo">
        ....
        <injection-target>
            <injection-target-class>
                com.foo.Bar.class
            </injection-target-class>
            <injection-target-name>
                baz
            </injection-target-name>
        </injection-target>
    </resource-ref>

CODE EXAMPLE 8-5

web.xml

    <resource-ref>
        <resource-ref-name="foo">
        ...
    </resource-ref>

Fragment 1

web-fragment.xml

    <resource-ref>
        <resource-ref-name="foo">
        ...
        <injection-target>
            <injection-target-class>
                com.foo.Bar.class
            </injection-target-class>
            <injection-target-name>
                baz
            </injection-target-name>
        </injection-target>
    </resource-ref>

Fragment 2

web-fragment.xml

    <resource-ref>
        <resource-ref-name="foo">
        ...
        <injection-target>
            <injection-target-class>
                com.foo.Bar2.class
            </injection-target-class>
            <injection-target-name>
                baz2
            </injection-target-name>
        </injection-target>
    </resource-ref>

有效的 metadata 是：

    <resource-ref>
        <resource-ref-name="foo">
        ....
        <injection-target>
            <injection-target-class>
                com.foo.Bar.class
            </injection-target-class>
            <injection-target-name>
                baz
            </injection-target-name>
        </injection-target>
        <injection-target>
            <injection-target-class>
                com.foo.Bar2.class
            </injection-target-class>
            <injection-target-name>
                baz2
            </injection-target-name>
        </injection-target>
    </resource-ref>

CODE EXAMPLE 8-6

web.xml

    <resource-ref>
        <resource-ref-name="foo">
        <injection-target>
            <injection-target-class>
              com.foo.Bar3.class
            </injection-target-class>
            <injection-target-name>
                baz3
            </injection-target-name>
        ...
    </resource-ref>

Fragment 1

web-fragment.xml

    <resource-ref>
        <resource-ref-name="foo">
        ...
        <injection-target>
            <injection-target-class>
                com.foo.Bar.class
            </injection-target-class>
            <injection-target-name>
                baz
            </injection-target-name>
        </injection-target>
    </resource-ref>

Fragment 2

web-fragment.xml

    <resource-ref>
        <resource-ref-name="foo">
        ...
        <injection-target>
            <injection-target-class>
                com.foo.Bar2.class
            </injection-target-class>
            <injection-target-name>
                baz2
            </injection-target-name>
        </injection-target>
    </resource-ref>

有效的 metadata 是：

    <resource-ref>
        <resource-ref-name="foo">
        <injection-target>
            <injection-target-class>
                com.foo.Bar3.class
            </injection-target-class>
            <injection-target-name>
                baz3
            </injection-target-name>
            <injection-target-class>
                com.foo.Bar.class
            </injection-target-class>
            <injection-target-name>
                baz
            </injection-target-name>
            <injection-target-class>
                com.foo.Bar2.class
            </injection-target-class>
            <injection-target-name>
                baz2
            </injection-target-name>
        </injection-target>
        ...
    </resource-ref>

Fragment 1和2的`<injection-target>`将被合并到主 web.xml

    * 如果主web.xml没有指定任何`<post-construct>`元素，且web-fragment中也指定了`<post-construct>` ，那么fragment中的  `<post-construct>`将被合并到主web.xml。不过如果在主web.xml中至少指定一个`<post-construct>`元素，那么fragment中的`<post-construct>`将不被合并。由web.xml的作者负责确保`<post-construct>`列表是完成的。
    * 如果主web.xml没有指定任何`<pre-destroy>`元素，且web-fragment中也指定了`<pre-destroy>`，那么fragment中的`<pre-destroy>`元素将被合并到主web.xml。不过如果在主web.xml中至少指定一个`<pre-destroy>`元素，那么fragment中的`<pre-destroy>`将不被合并。由web.xml的作者负责确保`<pre-destroy>`列表是完成的。
    * 在处理完web-fragment.xml之后，在处理下一个fragment之前相应fragment的注解被处理以完成有效的metadata。以下规则用于处理注解：
    * 通过注解指定的metadata，尚未存在于描述符中，将被用来扩充有效的描述符。
      * 指定在主web.xml或web fragment中的配置比通过注解指定的配置具有更高优先级。
      * 使用@WebServlet 注解定义Servlet，要使用描述符覆盖其值，描述符中的servlet名字必须匹配使用注解指定的servlet名字（明确指定或如果注解没有指定则是默认名字）。
      * 使用注解定义的Servlet和Filter初始化参数，如果描述符中的初始化参数的名字完全匹配指定在注解中的名字，则将被描述符中的覆盖。初始化参数在注解和描述符之间是可添加的。
      * url-pattern，当以给定servlet名字指定在描述符中时，将覆盖注解指定的url pattern。
      * 使用@WebFilter 注解定义的Filter，要使用描述符覆盖其值，描述符中的Filter名字必须匹配使用注解指定的Filter名字（明确指定或如果注解没有指定则是默认名字）。
      * Filter应用的url-pattern，当以给定Filter名字指定在描述符中时，将覆盖注解指定的url pattern。
      * Filter应用的DispatcherType，当以给定Filter名字指定在描述符中时，将覆盖注解指定的DispatcherType。
      * 下面的例子演示了上面的一些规则：

使用注解声明Servlet和在打包到的相应web.xml描述符中声明Servlet：

    @WebServlet(urlPatterns=”/MyPattern”, initParams=
    {@WebInitParam(name="ccc", value="333")})
    public class com.acme.Foo extends HttpServlet
    {
      ...
    }

web.xml

    <servlet>
        <servlet-class>com.acme.Foo</servlet-class>
        <servlet-name>Foo</servlet-name>
        <init-param>
            <param-name>aaa</param-name>
            <param-value>111</param-value>
        </init-param>
    </servlet>
    <servlet>
        <servlet-class>com.acme.Foo</servlet-class>
        <servlet-name>Fum</servlet-name>
        <init-param>
            <param-name>bbb</param-name>
            <param-value>222</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>Foo</servlet-name>
        <url-pattern>/foo/*</url-pattern>
    </servlet-mapping>
    <servlet-mapping>
        <servlet-name>Fum</servlet-name>
        <url-pattern>/fum/*</url-pattern>
    </servlet-mapping>

因为使用注解声明的 Servlet 名字不匹配在 web.xml 中声明的 servlet 名字，在 web.xml 中除了其他的声明外，注解指定一个新的servlet 声明，相当于：

    <servlet>
        <servlet-class>com.acme.Foo</servlet-class>
        <servlet-name>com.acme.Foo</servlet-name>
        <init-param>
            <param-name>ccc</param-name>
            <param-value>333</param-name>
    </servlet>

如果上面的web.xml被替换为如下：

    <servlet>
        <servlet-class>com.acme.Foo</servlet-class>
        <servlet-name>com.acme.Foo</servlet-name>
        <init-param>
            <param-name>aaa</param-name>
            <param-value>111</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>com.acme.Foo</servlet-name>
        <url-pattern>/foo/*</url-pattern>
    </servlet-mapping>

那么有效的描述符将等价于：

        <servlet-class>com.acme.Foo</servlet-class>
        <servlet-name>com.acme.Foo</servlet-name>
        <init-param>
            <param-name>aaa</param-name>
            <param-value>111</param-value>
        </init-param>
        <init-param>
            <param-name>ccc</param-name>
            <param-value>333</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>com.acme.Foo</servlet-name>
        <url-pattern>/foo/*</url-pattern>
    </servlet-mapping>

### 共享库 / 运行时可插拔性
除了支持fragment和使用注解的外，要求之一是我们不仅能plug-in 绑定在WEB-INF/lib下的，也能plugin框架共享副本—包括能plug-in到容器的如建议在web容器之上的JAX-WS、JAX-RS和JSF。
ServletContainerInitializer允许处理这样的使用情况下，如下所述。
ServletContainerInitializer类通过jar services API查找。对于每一个应用，应用启动时，由容器创建一个ServletContainerInitializer实例。框架提供的ServletContainerInitializer实现必须绑定在jar包的META-INF/services目录中的一个叫做javax.servlet.ServletContainerInitializer的文件，根据jar services API，指定ServletContainerInitializer的实现。

除ServletContainerInitializer外，我们还有一个注解—HandlesTypes。在ServletContainerInitializer 实现上的HandlesTypes注解用于表示感兴趣的一些类，它们可能指定了HandlesTypes的value中的注解（类型、方法或自动级别的注解），或者是其类型的超类继承/实现了这些类之一。无论是否设置了metadata-complete，HandlesTypes注解将应用。

当检测一个应用的类看是否它们匹配ServletContainerInitializer的HandlesTypes指定的条件时，如果应用的一个或多个可选的JAR包缺失，容器可能遇到类装载问题。由于容器不能决定是否这些类型的类装载失败将阻止应用正常工作，它必须忽略它们，同时也提供一个将记录它们的配置选项。

如果ServletContainerInitializer实现没有@HandlesTypes注解，或如果没有匹配任何指定的HandlesType，那么它会为每个应用使用null 值的集合调用一次。这将允许initializer基于应用中可用的资源决定是否需要初始化Servlet/Filter。

在任何Servlet Listener的事件被触发之前，当应用正在启动时，ServletContainerInitializer的onStartup方法将被调用。
ServletContainerInitializer’s的onStartup得到一个类的Set，其或者继承/实现initializer表示感兴趣的类，或者它是使用指定在@HandlesTypes注解中的任意类注解的。

下面一个具体的例子展示了这是如何工作的。

让我们学习 JAX-WS web service 运行时。

JAX-WS 运行时实现通常不是绑定到每个war包。其实现将绑定一个ServletContainerInitializer 的实现（如下所示）且容器将查找使用的services API（绑定在 jar 包中的 META-INF/services 目录中的一个叫做 javax.servlet.ServletContainerInitializer的文件，它将指出如下所示的 JAXWSServletContainerInitializer）。


    @HandlesTypes(WebService.class)
    JAXWSServletContainerInitializer
    implements ServletContainerInitializer
    {
    public void onStartup(Set<Class<?>> c, ServletContext ctx)
    throws ServletException {
        // JAX-WS specific code here to initialize the runtime
        // and setup the mapping etc.
        ServletRegistration reg = ctx.addServlet("JAXWSServlet",
        "com.sun.webservice.JAXWSServlet");
        reg.addServletMapping("/foo");
    }


框架的 jar 包也可能被绑定到war包目录中的 WEB-INF/lib 目录。如果ServletContainerInitializer 被绑定到应用的 WEB-INF/lib 目录内的一个 JAR 包中，它的 onStartup 方法在绑定到的应用启动期间将被仅调用一次。如果，相反，ServletContainerInitialzer 被绑定到 WEB-INF/lib 目录外的一个JAR包中，但仍能被运行时的服务提供商查找机制发现时，每次启动应用时，它的 onStartup 方法将被调用。

ServletContainerInitializer 接口的实现将被运行时的服务查找机制或语义上与它等价的容器特定机制发现。在任一种情况，web fragment JAR 包的 ServletContainerInitializer 服务被排除于一个 absolute ordering 必须被忽略，这些服务被发现的顺序必须遵照应用的类装载委托模型。
