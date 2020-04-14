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
1. 初始化 地址映射 代理类创建 地址规则匹配
2. AbstractHandlerMethodMapping系列是将Method作为Handler来使用的，这也是我们现在用得最多的一种Handler
3. @RequestMapping 类注解和方法上的注解 组合 RequestMappingHandlerMapping#getMappingForMethod
```
@Override
@Nullable
protected RequestMappingInfo getMappingForMethod(Method method, Class<?> handlerType) {
    RequestMappingInfo info = createRequestMappingInfo(method);
    if (info != null) {
        RequestMappingInfo typeInfo = createRequestMappingInfo(handlerType);
        if (typeInfo != null) {
            info = typeInfo.combine(info);
        }
        String prefix = getPathPrefix(handlerType);
        if (prefix != null) {
            info = RequestMappingInfo.paths(prefix).options(this.config).build().combine(info);
        }
    }
    return info;
}
```
4. AntPathMatcher#doMatch 处理url通配符模式配置  
5. ConversionServiceExposingInterceptor 配置定义的一个类型转换服务。该类型转换服务会在请求处理过程中用于请求参数或者返回值的类型转换。
```
@Override
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
        throws ServletException, IOException {

    request.setAttribute(ConversionService.class.getName(), this.conversionService);
    return true;
}
EvalTag#getConversionService
@Nullable
private ConversionService getConversionService(PageContext pageContext) {
    return (ConversionService) pageContext.getRequest().getAttribute(ConversionService.class.getName());
}
```
6. ResourceUrlProviderExposingInterceptor 配置定义的一个资源URL提供者对象ResourceUrlProvider
7. UriTemplateVariablesHandlerInterceptor 配置定义的一个资源URL上的参数存储
```
@Override
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
exposeUriTemplateVariables(this.uriTemplateVariables, request);
return true;
}
protected void exposeUriTemplateVariables(Map<String, String> uriTemplateVariables, HttpServletRequest request) {
    request.setAttribute(URI_TEMPLATE_VARIABLES_ATTRIBUTE, uriTemplateVariables);
}
解析url上的参数 PathVariableMethodArgumentResolver#resolveName
@Override
@SuppressWarnings("unchecked")
@Nullable
protected Object resolveName(String name, MethodParameter parameter, NativeWebRequest request) throws Exception {
    Map<String, String> uriTemplateVars = (Map<String, String>) request.getAttribute(
            HandlerMapping.URI_TEMPLATE_VARIABLES_ATTRIBUTE, RequestAttributes.SCOPE_REQUEST);
    return (uriTemplateVars != null ? uriTemplateVars.get(name) : null);
}   
```

小结 : 很多时候  会先把数据初始化到map中，然后通过key 查询对应的处理类，然后操作


## spring ioc容器初始化 
0. xml -> beandefinition
1. beandefinition -> bean instance
1. BeanDefinition 初始化
2. bean 创建初始化
3. factoryBean 

3. AliasRegistry 管理别名  但是还没弄清楚别名的作用
4. BeanDefinition用来定义 Bean的作用范围、角色、依赖、懒加载等与 Spring容器运行和管理Bean息息相关的属性，以达到对Bean的Spring特性的定制，是Spring Bean描述定义信息的核心接口类

5. Autowired AutowiredAnnotationBeanPostProcessor 当Spring容器启动时，AutowiredAnnotationBeanPostProcessor 将扫描 Spring 容器中所有 Bean，当发现 Bean 中拥有@Autowired 注释时就找到和其匹配（默认按类型匹配）的 Bean，并注入到对应的地方中去。  

## todo-list
1. spring 获取 某个interface的所有实例 例子
```
Map<String, HandlerMapping> matchingBeans = BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
```
2. 通过扫描一个class 获取 class method paramter 注解 
    - 2.1 searchWithFindSemantics 寻找策略
          searchWithFindSemantics 函数提供的方法可以防止无止尽的循环寻找注解，使用一个 visit 列表，将访问过的 element 保存在该列表中，保证 element 不会被循环寻找。
          
          searchWithFindSemantics 在四个地方查找是否有注解：
          当前 element 是否有注解(不考虑 @Inherited，即不考虑注解的继承)
          如果 element 是方法类型，即 element instanceof Method == true，在定义该 element 的接口中查找注解
          如果 element 是方法类型，从该类的直接父类开始，依次在父类中查找该方法的父方法，判断是否有该注解。
          如果 element 是类类型，那么会依次查找该类及其父类，看有没有注解。
3. mvc 参数解析处理器
4. 不同url对应不同的resource,不同的resource使用不同handler来读取

## jdk
1. AnnotatedElement 接口是所有程序元素（如Class、Method、Constructor等）的父接口 可以用该接口做注解操作
2. @FunctionalInterface 还可以怎么定义
```
public static final MethodFilter USER_DECLARED_METHODS = (method -> !method.isBridge() && !method.isSynthetic());

@FunctionalInterface
public interface MethodFilter {

    /**
     * Determine whether the given method matches.
     * @param method the method to check
     */
    boolean matches(Method method);
}

```

