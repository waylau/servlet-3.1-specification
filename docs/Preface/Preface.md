前言
====

本文档是 Java™ Servlet 规范，针对版本是 3.1。本文档描述了 Java Servlet API 的标准。

## 其他资料

本规范制定的目的是给 Java Servlet 一个完整和清晰的解释。如果有仍有问题，可以查阅以下资料：

* 一个参考实现（简称 RI）：已经实现并提供了本规范的行为基准。该参考实现没有对一个详细的特性实现去诠释，其他实现者可以以参考实现作为原型，以此原型完成规范。
* 一个兼容性测试套件（简称 CTS）：用来验证实现是否兼容 Java Servlet API 标准需求。并且测试结果为分析一个实现是不是标准实现提供了一个规范值。
* 如果需要进一步澄清疑问，可以咨询 Java Community Process(Java 社区进程，简称JCP）控制下的 Java Servlet API 工作组，他们是问题的最终判定者。

非常欢迎建议和反馈，这些信息可以用来改善未来版本。

## 谁应该读此规范

本规范的目标读者有如下几种：

* Web 服务器和应用服务器供应商，用于开发符合此标准的 servlet 引擎。
* 工具开发者，想要开发符合此规范的 Web 应用的支持工具。
* 有经验的 servlet 开发者，想要理解 servlet 技术的底层机制。

该规范不是 servlet 开发人的用户指南，而且也并不打算被用作这样。用于此目的参考文献可以到<http://java.sun.com/products/servlet>查找。

## API 规范

定义了 Java Servlet API 中类、接口、方法签名的完整规范，且附带的Javadoc 文档有可用的在线版。

## 其他的 Java 平台规范

该规范参考如下其他 Java API 规范：

* Java Platform, Enterprise Edition ("Java EE"), version 7
* JavaServer Pages™ ("JSP™"), version 2.2
* Java Naming and Directory Interface™ ("J.N.D.I.").
* Context and Dependency Injection for the Java EE Platform
* Managed Beans specification

这些规范可以在 Java Platform, Enterprise Edition 网站中找到：<http://java.sun.com/javaee/>。

其他重要参考资料
以下Internet规范提供了一些有关开发和实现Java Servlet API和标准servlet引擎的信息：

* RFC 1630 Uniform Resource Identifiers (URI)
* RFC 1738 Uniform Resource Locators (URL)
* RFC 2396 Uniform Resource Identifiers (URI): Generic Syntax
* RFC 1808 Relative Uniform Resource Locators
* RFC 1945 Hypertext Transfer Protocol (HTTP/1.0)
* RFC 2045 MIME Part One: Format of Internet Message Bodies
* RFC 2046 MIME Part Two: Media Types
* RFC 2047 MIME Part Three: Message Header Extensions for non-ASCII text
* RFC 2048 MIME Part Four: Registration Procedures
* RFC 2049 MIME Part Five: Conformance Criteria and Examples
* RFC 2109 HTTP State Management Mechanism
* RFC 2145 Use and Interpretation of HTTP Version Numbers
* RFC 2324 Hypertext Coffee Pot Control Protocol (HTCPCP/1.0)1
* RFC 2616 Hypertext Transfer Protocol (HTTP/1.1)
* RFC 2617 HTTP Authentication: Basic and Digest Authentication
* RFC 3986 Uniform Resource Identifier (URI): Generic Syntax

RFC 在线版本请访问：<http://wwww.ietf.org/rfc/>。

万维网联盟（<http://www.w3.org/>）是影响本规范和实现的 HTTP 相关来源信息的权威。

可扩展的标记语言（XML）：用于此规范第13章描述的部署描述符。更多的XML 信息可以在以下网站找到：

<http://java.sun.com/xml>

<http://www.xml.org/>

## 提供反馈
我们欢迎大家提供此规范的任意和所有的反馈。请发送你的建议到users@servlet-spec.java.net 邮箱。

请注意，由于我们收到大量的反馈意见，你可能不能正常收到来自工程师的回复。尽管如此，规范团队会阅读、评估、存档每一个建议。

## 专家组成员

* Deepak Anupalli (Pramati Technologies)
* Euigeun Chung (TmaxSoft, Inc)
* Robert Goff (IBM)
* Richard Hightower
* Seth Hodgson (Adobe Systems Inc.)
* Remy Maucherat (RedHat)
* Minoru Nitta (Fujitsu Limited)
* Ramesh PVK (Pramati Technnologies)
* Alex Rojkov (Caucho Technologies)
* Mark Thomas (VMware)
* Gregory John Wilkins
* Wenbo Zhu (Google Inc.)

## 答谢

Oracle 的 Bill Shannon 为该规范提供了非常宝贵的技术投入。Oracle的 Ron Monzillo 帮助推动了一些建议和围绕安全方面的技术讨论。