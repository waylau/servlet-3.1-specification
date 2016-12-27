属性
====

属性是与请求相关联的对象。属性可以由容器设置来表达信息，否则无法通过 API 表示，或者由 servlet 设置将信息传达给另一个 servlet（通过 RequestDispatcher）。属性通过 ServletRequest 接口中下面的方法来访问：

* getAttribute
* getAttributeNames
* setAttribute

一个属性名称只能关联一个属性值。

前缀 java. 和 javax. 开头的属性名称是本规范的保留定义。同样地，前缀 sun. 和 com.sun.，oracle 和 com.oracle 开头的属性名是Oracle Corporation 的保留定义。建议属性集中所有属性的命名与[Java编程语言的规范](http://docs.oracle.com/javase/specs/)为包命名建议的反向域名约定一致。
