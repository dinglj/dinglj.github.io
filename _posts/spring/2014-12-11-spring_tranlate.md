###spring 翻译[ioc-依赖注入控制器]
http://docs.spring.io/spring/docs/3.2.x/spring-framework-reference/htmlsingle/#overview-core-container
1.2.1 核心容器
spring核心容器是有core,bean,context和expression language模块组成
core和beans模块是整个spring框架最基础的部分，包括ioc和依赖注入的特性，BeanFactory是一个工厂模式的复杂实现，帮你实现了单例模式，通过配置把实际程序的的依赖进行解构
context模块是在core和beans的模块上开发的，这意味着在框架式中访问对象类似于jndi的注册，context模块继承了beans模块的特征并且支持国际化功能...
expression language模块提供了强大的表达式语言查询和操作一个运行时的对象图表，还拓展了同一表达式语言(el)作为jsp 2.1的规范，该语言支持设置(setting)获取(getting)属性值，属性赋值，方法调用，访问context中的数组，集合和索引，逻辑和算术操作，变量和被ioc容器命名的回收对象..
1.2.2数据访问/整合
由jdbc,orm,oxm,jms和transaction模块组成
jdbc模块提供了提供jdbc抽象层减少手动操作jdbc和解析数据库供应商的错误码
orm模块整合了比较流行的对象关系映射框架 包括jpa,jdo,hibernate和ibatis，使用orm包你可以使用整合了所有的O/R映射框架的特性，比如声明式事务管理（declarative transaction management）特性
oxm模块提供支持对象和xml映射可通过jaxb,castor,xmlbeans,jibx和xstream的实现
transaction模块支持...


