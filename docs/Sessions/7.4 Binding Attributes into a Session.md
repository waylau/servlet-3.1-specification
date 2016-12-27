绑定属性到会话
====

servlet 可以按名称绑定对象属性到 HttpSession 实现，任何绑定到会话的对象可用于任意其他的 servlet，其属于同一个 ServletContext 且处理属于相同会话中的请求。
一些对象可能需要在它们被放进会话或从会话中移除时得到通知。这些信息可以从 HttpSessionBindingListener 接口实现的对象中获取。这个接口定义了以下方法，用于标识一个对象被绑定到会话或从会话解除绑定时。

* valueBound
* valueUnbound

在对象对 HttpSession 接口的 getAttribute 方法可用之前 valueBound 方法必须被调用。在对象对 HttpSession 接口的 getAttribute 方法不可用之后 valueUnbound 方法必须被调用。
