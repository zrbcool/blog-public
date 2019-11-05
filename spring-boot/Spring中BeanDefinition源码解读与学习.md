- parentName  
  父BD的名字（如果有的话）
- beanClassName  
  BD所定义的Bean的具体Class的名字
- scope  
  spring中的作用域，只有两种情况SCOPE_SINGLETON，SCOPE_PROTOTYPE即单例和原型两种
- lazyInit  
  是否延迟初始化
- dependsOn[]  
  指定依赖的beanName，beanFactory会保证依赖优先全部初始化
- autowireCandidate  
  注明这个bean是AutoWired的候选，但是仅当type-based autowire时有效，对使用name进行autowire的情况无任何影响
- primary  
  注明这个bean是AutoWire的首选（Primary）Bean
- factoryBeanName  
  设置这个bean的FactoryBean的名字（如果有的话）
- factoryMethodName  
  与上个参数有关