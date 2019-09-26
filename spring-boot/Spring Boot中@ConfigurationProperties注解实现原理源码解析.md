## 0. 开源项目推荐
[Pepper Metrics](https://github.com/zrbcool/pepper-metrics)是我与同事开发的一个开源工具(https://github.com/zrbcool/pepper-metrics)，其通过收集jedis/mybatis/httpservlet/dubbo/motan的运行性能统计，并暴露成prometheus等主流时序数据库兼容数据，通过grafana展示趋势。其插件化的架构也非常方便使用者扩展并集成其他开源组件。  
请大家给个star，同时欢迎大家成为开发者提交PR一起完善项目。

## 1. 概述
不用说大家都知道Spring Boot非常的方便，快捷，让开发的同学简单的几行代码加上几行配置甚至零配置就能启动并使用一个项目，项目当中我们也可能经常使用
@ConfigurationProperties将某个Bean与properties配置当中的prefix相绑定，使配置值与定义配置的Bean分离，方便管理。  
那么，这个@ConfigurationProperties是什么机制，如何实现的呢？我们今天来聊聊这个话题

## 2. 正文
### 2.1 从EnableConfigurationProperties说起
为什么从EnableConfigurationProperties讲？
Spring Boot项目自身当中大量autoconfigure都是使用EnableConfigurationProperties注解启用XXXProperties功能，例如spring-data-redis的
这个RedisAutoConfiguration
```java
@Configuration
@ConditionalOnClass(RedisOperations.class)
@EnableConfigurationProperties(RedisProperties.class) //看这里
@Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
public class RedisAutoConfiguration {
    // ...
}
```
而RedisProperties中又带有注解@ConfigurationProperties(prefix = "spring.redis")，这样就将spring.redis这个前缀的配置项与RedisProperties
这个实体类进行了绑定。  

### 2.2 EnableConfigurationProperties内部实现解析
说完来由，我们就来说说内部实现，先来看看
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(EnableConfigurationPropertiesImportSelector.class)
public @interface EnableConfigurationProperties {
	Class<?>[] value() default {};
}
```
@Import(EnableConfigurationPropertiesImportSelector.class)指明了这个注解的处理类EnableConfigurationPropertiesImportSelector，
查看EnableConfigurationPropertiesImportSelector源码
```java
class EnableConfigurationPropertiesImportSelector implements ImportSelector {

	private static final String[] IMPORTS = { ConfigurationPropertiesBeanRegistrar.class.getName(),
			ConfigurationPropertiesBindingPostProcessorRegistrar.class.getName() };

	@Override
	public String[] selectImports(AnnotationMetadata metadata) {
		return IMPORTS;
	}
	
	// 省略部分其他方法
}
```
我们先看这块关键部分返回了一个IMPORTS数组，数组中包含ConfigurationPropertiesBeanRegistrar.class，ConfigurationPropertiesBindingPostProcessorRegistrar.class两个元素  
根据@Import及ImportSelector接口的原理（其原理可以参考同事写的一篇文章：[相亲相爱的@Import和@EnableXXX](https://zhuanlan.zhihu.com/p/83295224)），我们得知spring会初始化上面两个Registrar到spring容器当中，而两个Registrar均实现了ImportBeanDefinitionRegistrar接口，
而ImportBeanDefinitionRegistrar会在处理Configuration时触发调用（其原理可以参考文章：[这块找一篇文章](https://blog.wangqi.love/articles/Java/%E5%8A%A8%E6%80%81%E6%B3%A8%E5%86%8Cbean(ImportBeanDefinitionRegistrar,%20FactoryBean).html)），下面我们分别深入两个Registrar的源码：
- ConfigurationPropertiesBeanRegistrar
- ConfigurationPropertiesBindingPostProcessorRegistrar
#### 2.2.1 ConfigurationPropertiesBindingPostProcessorRegistrar
直接看代码
```java
public class ConfigurationPropertiesBindingPostProcessorRegistrar implements ImportBeanDefinitionRegistrar {

	@Override
	public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
		if (!registry.containsBeanDefinition(ConfigurationPropertiesBindingPostProcessor.BEAN_NAME)) {
			registerConfigurationPropertiesBindingPostProcessor(registry);
			registerConfigurationBeanFactoryMetadata(registry);
		}
	}

	private void registerConfigurationPropertiesBindingPostProcessor(BeanDefinitionRegistry registry) {
		GenericBeanDefinition definition = new GenericBeanDefinition();
		definition.setBeanClass(ConfigurationPropertiesBindingPostProcessor.class);
		definition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
		registry.registerBeanDefinition(ConfigurationPropertiesBindingPostProcessor.BEAN_NAME, definition);
	}

	private void registerConfigurationBeanFactoryMetadata(BeanDefinitionRegistry registry) {
		GenericBeanDefinition definition = new GenericBeanDefinition();
		definition.setBeanClass(ConfigurationBeanFactoryMetadata.class);
		definition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
		registry.registerBeanDefinition(ConfigurationBeanFactoryMetadata.BEAN_NAME, definition);
	}
}
```
可以看到注册了两个Bean到spring容器
- ConfigurationPropertiesBindingPostProcessor
    - 其实现如下接口：
      BeanPostProcessor, PriorityOrdered, ApplicationContextAware, InitializingBean
      - PriorityOrdered  
        Ordered.HIGHEST_PRECEDENCE + 1保证前期执行，且非最先
      - ApplicationContextAware  
        获取到ApplicationContext设置到内部变量
      - InitializingBean  
        afterPropertiesSet方法在Bean创建时被调用，保证内部变量configurationPropertiesBinder被初始化，这个binder类就是使prefix与propertyBean进行值绑定的关键工具类
      - BeanPostProcessor
        postProcessBeforeInitialization方法处理具体的bind逻辑如下：
```java
      @Override
      public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
          ConfigurationProperties annotation = getAnnotation(bean, beanName, ConfigurationProperties.class);
          if (annotation != null) {
              bind(bean, beanName, annotation);
          }
          return bean;
      }
      private void bind(Object bean, String beanName, ConfigurationProperties annotation) {
          ResolvableType type = getBeanType(bean, beanName);
          Validated validated = getAnnotation(bean, beanName, Validated.class);
          Annotation[] annotations = (validated != null) ? new Annotation[] { annotation, validated }
                  : new Annotation[] { annotation };
          Bindable<?> target = Bindable.of(type).withExistingValue(bean).withAnnotations(annotations);
          try {
              // 在这里完成了，关键的prefix到PropertyBean的值绑定部分，所以各种@ConfigurationProperties注解最终生效就靠这部分代码了
              this.configurationPropertiesBinder.bind(target);
          }
          catch (Exception ex) {
              throw new ConfigurationPropertiesBindException(beanName, bean, annotation, ex);
          }
      }
```
- ConfigurationBeanFactoryMetadata  
  如果某些Bean是通过FactoryBean创建，则该类用于保存FactoryBean的各种原信息，用于ConfigurationPropertiesBindingPostProcessor当中的元数据查询，这里就不做展开

#### 2.2.2 ConfigurationPropertiesBeanRegistrar
其实ConfigurationPropertiesBeanRegistrar是EnableConfigurationPropertiesImportSelector的静态内部类，在前面贴代码时被省略的部分，上代码
```java
	public static class ConfigurationPropertiesBeanRegistrar implements ImportBeanDefinitionRegistrar {

		@Override
		public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
			getTypes(metadata).forEach((type) -> register(registry, (ConfigurableListableBeanFactory) registry, type));
		}

		private List<Class<?>> getTypes(AnnotationMetadata metadata) {
			MultiValueMap<String, Object> attributes = metadata
					.getAllAnnotationAttributes(EnableConfigurationProperties.class.getName(), false);
			return collectClasses((attributes != null) ? attributes.get("value") : Collections.emptyList());
		}

		private List<Class<?>> collectClasses(List<?> values) {
			return values.stream().flatMap((value) -> Arrays.stream((Object[]) value)).map((o) -> (Class<?>) o)
					.filter((type) -> void.class != type).collect(Collectors.toList());
		}

		private void register(BeanDefinitionRegistry registry, ConfigurableListableBeanFactory beanFactory,
				Class<?> type) {
			String name = getName(type);
			if (!containsBeanDefinition(beanFactory, name)) {
				registerBeanDefinition(registry, name, type);
			}
		}

		private String getName(Class<?> type) {
			ConfigurationProperties annotation = AnnotationUtils.findAnnotation(type, ConfigurationProperties.class);
			String prefix = (annotation != null) ? annotation.prefix() : "";
			return (StringUtils.hasText(prefix) ? prefix + "-" + type.getName() : type.getName());
		}

		private void registerBeanDefinition(BeanDefinitionRegistry registry, String name, Class<?> type) {
			assertHasAnnotation(type);
			GenericBeanDefinition definition = new GenericBeanDefinition();
			definition.setBeanClass(type);
			registry.registerBeanDefinition(name, definition);
		}

		private void assertHasAnnotation(Class<?> type) {...}
		private boolean containsBeanDefinition(ConfigurableListableBeanFactory beanFactory, String name) {...}
	}
```
逻辑解读：  
主逻辑入口（registerBeanDefinitions）  
1 -> getTypes(metadata)拿到标注EnableConfigurationProperties注解的配置值，整理成List<Class<?>>然后逐个处理  
2 -> 对每个元素（Class<?>）调用register方法处理  
3 -> register方法通过类当中的ConfigurationProperties注解的prefix值加类名字作为beanName通过registry.registerBeanDefinition调用将Class<?>注册到registry当中  
当标注注解@ConfigurationProperties的XXXProperties的BeanDefinition加入到registry中后，Bean的初始化就交给spring容器，
而这个过程中前面提到的ConfigurationPropertiesBindingPostProcessorRegistrar就完成一系列的后置操作帮助我们完成最终的值绑定

## 3. 总结
@ConfigurationProperties的整体处理过程，本文已经基本讲述完毕，现在大体总结一下：
EnableConfigurationProperties完成ConfigurationPropertiesBindingPostProcessorRegistrar及ConfigurationPropertiesBeanRegistrar的引入
其中：
- ConfigurationPropertiesBeanRegistrar完成标注@ConfigurationProperties的类的查找并组装成BeanDefinition加入registry
- ConfigurationPropertiesBindingPostProcessorRegistrar完成ConfigurationPropertiesBindingPostProcessor及ConfigurationBeanFactoryMetadata
    - ConfigurationPropertiesBindingPostProcessor完成所有标注@ConfigurationProperties的Bean到prefix的properties值绑定
    - ConfigurationBeanFactoryMetadata仅用于提供上面处理中需要的一些元数据信息
## 4. 作者其他文章
[https://github.com/zrbcool/blog-public](https://github.com/zrbcool/blog-public)  

## 5. 微信订阅号
![](http://oss.zrbcool.top/Fv816XFbZB2JQazo5LHBoy2_SGVz)