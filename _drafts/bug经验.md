### Spring AOP 失效问题

参考：

> https://mp.weixin.qq.com/s/PtRN_YoDmIj0tLWc-uKBOw

类中的方法 A 调用本类方法 B，A 中定义所切的方法生效，但 B 所定义的方法切不到，需要 A 获取到所在类的代理对象。

