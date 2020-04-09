---
layout : post
title : "spring 知识点"
category : spring
duoshuo: true
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
1. aware 英译过来:知道的，已感知的，意识到的 通俗点解释就是：如果在某个类里面想要使用spring的一些东西，就可以通过实现XXXAware接口告诉spring，spring看到后就会给你送过来，而接收的方式是通过实现接口唯一的方法set-XXX。比如，有一个类想要使用当前的ApplicationContext，那么我们只需要让它实现ApplicationContextAware接口，然后实现接口中唯一的方法void setApplicationContext（ApplicationContextapplicationContext）就可以了
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
        	
        private void invokeAwareInterfaces(Object bean) {
                if (bean instanceof Aware) {
                    if (bean instanceof EnvironmentAware) {
                        ((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
                    }
                    if (bean instanceof EmbeddedValueResolverAware) {
                        ((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
                    }
                    if (bean instanceof ResourceLoaderAware) {
                        ((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
                    }
                    if (bean instanceof ApplicationEventPublisherAware) {
                        ((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
                    }
                    if (bean instanceof MessageSourceAware) {
                        ((MessageSourceAware) bean).setMessageSource(this.applicationContext);
                    }
                    if (bean instanceof ApplicationContextAware) {
                        ((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
                    }
                }
            }
```
2. xxCapable 具有某种xx能力 EnvironmentCapable 具有获取Environment能力
```
    public interface EnvironmentCapable {
    
        /**
         * Return the {@link Environment} associated with this component.
         */
        Environment getEnvironment();
    
    }
```
3. BeanWrapper Spring提供的一个用来操作JavaBean属性的工具
```
    BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
    ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
    bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
    initBeanWrapper(bw);
    bw.setPropertyValues(pvs, true);
```
4. MultiValueMap 一键多值 通常会自己定义 Map<K, List<V>>


### spring xml 命名空间解析处理类
1. mvc命名空间的解析设置在spring-webmvc-4.1.5.RELEASE.jar包下META-INF/spring.handlers文件中



### spring mvc 流程记录
