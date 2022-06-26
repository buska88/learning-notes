# jvm相关

## 1 工具

### 1.1 jmx

https://www.liaoxuefeng.com/wiki/1252599548343744/1282385687609378

https://www.oracle.com/technical-resources/articles/javase/jmx.html

https://www.cnblogs.com/dongguacai/p/5900507.html

JMX是Java Management Extensions，它是一个Java平台的管理和监控接口，对运行时jvm的系统信息进行监控。Java平台使用JMX作为管理和监控的标准接口，任何程序，只要按JMX规范访问这个接口，就可以获取所有管理与监控信息。

JMX把所有被管理的资源都称为MBean（Managed Bean），这些MBean全部由MBeanServer管理，如果要访问MBean，可以通过MBeanServer对外提供的访问接口，例如通过RMI或HTTP访问。

比如要访线程信息，就可以使用ThreadMXBean。