---
layout : post
title : "spring 知识点"
category : spring
duoshuo:true
date : 2020-04-08
tags : [spring,注解]
---
### spring核心功能
1. 控制反转: 业务实现往往是多个组件的之间合作完成，这就需要组件间存在依赖关系，如果获取组件的引用靠具体对象的来控制，这会导致高度的耦合，如果把这种依赖关系的引用交由专门的容器来管理是非常有价值的
2. 依赖注入: 

### spring-boot 注解
1. @SpringBootApplication 复合注解 @SpringBootConfiguration + @EnableAutoConfiguration + @ComponentScan
2. @SpringBootConfiguration -> @Configuration 
    - 2.1 @Configuration : 注入当前类钟用@Bean的bean
3. @EnableAutoConfiguration -> 包含 @Import -> EnableAutoConfigurationImportSelector
	- 3.1 @Import : 把注解内的bean注入到当前的容器内
4. @ComponentScan : 组件扫描
5. @import : 把注解内的bean注入到当前的容器内
6. @ConditionalOnxx : 当满足xx条件时，才实例化该bean
7. @Configuration :  注入当前类钟用@Bean的bean



### spring 接口
1. aware 英译过来:知道的，已感知的，意识到的 实现该接口可以获取ioc容器中bean的id 在bean初始化的时候如下
    ```
        private void invokeAwareMethods(final String beanName, final Object bean) {
        		if (bean instanceof Aware) {
        			if (bean instanceof BeanNameAware) {
        				((BeanNameAware) bean).setBeanName(beanName);
        			}
        			if (bean instanceof BeanClassLoaderAware) {
        				ClassLoader bcl = getBeanClassLoader();
        				if (bcl != null) {
        					((BeanClassLoaderAware) bean).setBeanClassLoader(bcl);
        				}
        			}
        			if (bean instanceof BeanFactoryAware) {
        				((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
        			}
        		}
        	}
    ```
