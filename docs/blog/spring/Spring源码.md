<a name="x9TgT"></a>
## Spring如何解决循环依赖的
> [https://www.zhihu.com/question/438247718/answer/1730527725](https://www.zhihu.com/question/438247718/answer/1730527725)

<a name="dxLQF"></a>
#### spring内部有三级缓存：

- singletonObjects 一级缓存，用于保存实例化、注入、初始化完成的bean实例
- earlySingletonObjects 二级缓存，用于保存实例化完成但不完整 的bean实例
- singletonFactories 三级缓存，用于保存bean创建工厂，以便于后面扩展有机会创建代理对象，也可以说是spring一开始不知道循环依赖，这个可以标志bean在创建中出现循环依赖，这个map 才是打破循环的

![image.png](https://cdn.nlark.com/yuque/0/2021/png/2660467/1639885233152-70d1c38a-786c-45c7-b009-0f9add3a529a.png#clientId=u9bec8613-04a7-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=310&id=u02dd8f2b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=619&originWidth=1440&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1199267&status=done&style=none&taskId=u450ddb3d-f2c3-4926-8408-c7d54997484&title=&width=720)<br />放到二级缓存里的已经实例化后的对象了，三级缓存放入的是ObjectFactory对象<br />  从三级缓存获取实例后，添加到二级缓存，并把三级缓存里的删除<br />testService1 在第4步骤中，会进一步初始化，从二级缓存中获取不完整的bean,最终添加的一级缓存里
<a name="Vp2Lj"></a>
### bean初始化过程
![image.png](https://cdn.nlark.com/yuque/0/2022/png/2660467/1650287689409-8209d516-27db-4a9c-95f3-797e1e5ef29e.png#clientId=ue58e0ed8-a1c3-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=384&id=u602f2a30&margin=%5Bobject%20Object%5D&name=image.png&originHeight=768&originWidth=1272&originalType=binary&ratio=1&rotation=0&showTitle=false&size=413755&status=done&style=none&taskId=ua1327266-b532-44c3-a02d-7f9bd40cf04&title=&width=636)<br />初始化包含3个步骤顺序：@PostConsruct < InitializinBean<init-method<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/2660467/1650618448053-b87f2015-ea98-4125-818e-c52238225bea.png#clientId=u5322599a-66e1-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=790&id=u5561612f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=790&originWidth=1012&originalType=binary&ratio=1&rotation=0&showTitle=false&size=177108&status=done&style=none&taskId=u1c68d162-c2a8-4628-b63c-f9682774e1f&title=&width=1012)getBean<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/2660467/1650623559473-26fd1c7a-a361-4294-9b2e-e5ab93b4818c.png#clientId=u5322599a-66e1-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u21754b02&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1080&originWidth=1129&originalType=url&ratio=1&rotation=0&showTitle=false&size=357043&status=done&style=none&taskId=u31dd0e31-f335-4003-904e-cf527bec911&title=)
> UserService类
> 推断构造方法。     默认无参构造，多个有参要用@Autowired 标注
> 普通对象
> 依赖注入     填充属性
> 初始化前     @PostConstruct
> 初始化		IntitializingBean
> 初始化后       AOP  在之前发生循环依赖且产生代理对象，这里就不会执行
> 代理对象        
> 放入Map单例池
> Bean 对象

> BeanFactoryPostProcessor接口中只有一个方法postProcessBeanFactory，该方法在spring容器初始化后执行，并且只执行一次。它会在BeanPostProcessor中的方法执行之前先执行
> AutowiredAnnotationBeanPostProcessor：处理@Autowired、@Value
> CommonAnnotationBeanPostProcessor：处理@Resource、@PostConstruct、@PreDestroy
> @Value的处理器StringValueResolver初始化时机是PropertySourcesPlaceholderConfigurer(实现了BeanPostProcessor)#postProcessBeanFactory中，而处理@Value属性解析的时机是在getBean中的依赖处理
> resolveDependency方法中

<a name="QLftK"></a>
### ![image.png](https://cdn.nlark.com/yuque/0/2022/png/2660467/1650785937496-6e6fe9dc-98a8-4402-b7ae-7eb6260648d7.png#clientId=u5b59a88c-8c2b-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u6ff5ada3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=505&originWidth=1601&originalType=url&ratio=1&rotation=0&showTitle=false&size=64638&status=done&style=none&taskId=u54bc7377-9d2e-474a-9ed7-b588fddd9a0&title=) 
<a name="WUAmI"></a>
## Mybatis整合Spring
  1.  利用FactoryBean.getObject()产生UserMapper代理对象,代理对象就会加入单例池里

2. 利用BeanDefinition产生FactoryBean 时使用有参构造，传入需要代理的对象
2. 调用 applicationContext.registerBeanDefinition()注册bd, 下面优化
2. 实现ImportBeanDefinitionRegister ,使用扫描器扫描bean,结合@Import()注解完成bd注入 下面优化
2. 使用@MapperScan 扫描路径 获取所有mapper 


<a name="nXIZW"></a>
### @Lookup 
> 作用在方法上，会生成一个代理对象，查找这个bean


<a name="RqCub"></a>
## SpringIOC过程
尚硅谷完整记录
> [https://liayun.blog.csdn.net/article/details/115053350](https://liayun.blog.csdn.net/article/details/115053350)

<a name="TxsVz"></a>
#### 1. 前期准备过程
```java
Spring容器的refresh()【创建刷新】;
1、prepareRefresh()刷新前的预处理;
	1）、initPropertySources()初始化一些属性设置;子类自定义个性化的属性设置方法；
	2）、getEnvironment().validateRequiredProperties();检验属性的合法等
	3）、earlyApplicationEvents= new LinkedHashSet<ApplicationEvent>();保存容器中的一些早期的事件；
2、obtainFreshBeanFactory();获取BeanFactory；
	1）、refreshBeanFactory();刷新【创建】BeanFactory；
			创建了一个this.beanFactory = new DefaultListableBeanFactory();
			设置id；
	2）、getBeanFactory();返回刚才GenericApplicationContext创建的BeanFactory对象；
	3）、将创建的BeanFactory【DefaultListableBeanFactory】返回；
3、prepareBeanFactory(beanFactory);BeanFactory的预准备工作（BeanFactory进行一些设置）；
	1）、设置BeanFactory的类加载器、支持表达式解析器...
	2）、添加部分BeanPostProcessor【ApplicationContextAwareProcessor】
	3）、设置忽略的自动装配的接口EnvironmentAware、EmbeddedValueResolverAware、xxx；
	4）、注册可以解析的自动装配；我们能直接在任何组件中自动注入：
			BeanFactory、ResourceLoader、ApplicationEventPublisher、ApplicationContext
	5）、添加BeanPostProcessor【ApplicationListenerDetector】
	6）、添加编译时的AspectJ；
	7）、给BeanFactory中注册一些能用的组件；
		environment【ConfigurableEnvironment】、
		systemProperties【Map<String, Object>】、
		systemEnvironment【Map<String, Object>】
4、postProcessBeanFactory(beanFactory);BeanFactory准备工作完成后进行的后置处理工作；
	1）、子类通过重写这个方法来在BeanFactory创建并预准备完成以后做进一步的设置
======================以上是BeanFactory的创建及预准备工作==================================
```
<a name="roVTC"></a>
#### 2. 执行BeanFactoryPostProcessors
```java
5、invokeBeanFactoryPostProcessors(beanFactory);执行BeanFactoryPostProcessor的方法；
	BeanFactoryPostProcessor：BeanFactory的后置处理器。在BeanFactory标准初始化之后执行的；
	两个接口：BeanFactoryPostProcessor、BeanDefinitionRegistryPostProcessor
	1）、执行BeanFactoryPostProcessor的方法；
		先执行BeanDefinitionRegistryPostProcessor
		1）、获取所有的BeanDefinitionRegistryPostProcessor；
		2）、看先执行实现了PriorityOrdered优先级接口的BeanDefinitionRegistryPostProcessor、
			postProcessor.postProcessBeanDefinitionRegistry(registry)
		3）、在执行实现了Ordered顺序接口的BeanDefinitionRegistryPostProcessor；
			postProcessor.postProcessBeanDefinitionRegistry(registry)
		4）、最后执行没有实现任何优先级或者是顺序接口的BeanDefinitionRegistryPostProcessors；
			postProcessor.postProcessBeanDefinitionRegistry(registry)
			
		
		再执行BeanFactoryPostProcessor的方法
		1）、获取所有的BeanFactoryPostProcessor
		2）、看先执行实现了PriorityOrdered优先级接口的BeanFactoryPostProcessor、
			postProcessor.postProcessBeanFactory()
		3）、在执行实现了Ordered顺序接口的BeanFactoryPostProcessor；
			postProcessor.postProcessBeanFactory()
		4）、最后执行没有实现任何优先级或者是顺序接口的BeanFactoryPostProcessor；
			postProcessor.postProcessBeanFactory()

```
<a name="ggWjZ"></a>
#### 3. 注册BeanPostProcessors
```java
6、registerBeanPostProcessors(beanFactory);注册BeanPostProcessor（Bean的后置处理器）【 intercept bean creation】
		不同接口类型的BeanPostProcessor；在Bean创建前后的执行时机是不一样的
		BeanPostProcessor、
		DestructionAwareBeanPostProcessor、
		InstantiationAwareBeanPostProcessor、
		SmartInstantiationAwareBeanPostProcessor、
		MergedBeanDefinitionPostProcessor【internalPostProcessors】、
		
		1）、获取所有的 BeanPostProcessor;后置处理器都默认可以通过PriorityOrdered、Ordered接口来执行优先级
		2）、先注册PriorityOrdered优先级接口的BeanPostProcessor；
			把每一个BeanPostProcessor；添加到BeanFactory中
			beanFactory.addBeanPostProcessor(postProcessor);
		3）、再注册Ordered接口的
		4）、最后注册没有实现任何优先级接口的
		5）、最终注册MergedBeanDefinitionPostProcessor；
		6）、注册一个ApplicationListenerDetector；来在Bean创建完成后检查是否是ApplicationListener，如果是
			applicationContext.addApplicationListener((ApplicationListener<?>) bean);

```
<a name="Wp1MU"></a>
#### 4. 国际化，事件
```java
7、initMessageSource();初始化MessageSource组件（做国际化功能；消息绑定，消息解析）；
		1）、获取BeanFactory
		2）、看容器中是否有id为messageSource的，类型是MessageSource的组件
			如果有赋值给messageSource，如果没有自己创建一个DelegatingMessageSource；
				MessageSource：取出国际化配置文件中的某个key的值；能按照区域信息获取；
		3）、把创建好的MessageSource注册在容器中，以后获取国际化配置文件的值的时候，可以自动注入MessageSource；
			beanFactory.registerSingleton(MESSAGE_SOURCE_BEAN_NAME, this.messageSource);	
			MessageSource.getMessage(String code, Object[] args, String defaultMessage, Locale locale);
8、initApplicationEventMulticaster();初始化事件派发器；
		1）、获取BeanFactory
		2）、从BeanFactory中获取applicationEventMulticaster的ApplicationEventMulticaster；
		3）、如果上一步没有配置；创建一个SimpleApplicationEventMulticaster
		4）、将创建的ApplicationEventMulticaster添加到BeanFactory中，以后其他组件直接自动注入
9、onRefresh();留给子容器（子类）
		1、子类重写这个方法，在容器刷新的时候可以自定义逻辑；
10、registerListeners();给容器中将所有项目里面的ApplicationListener注册进来；
		1、从容器中拿到所有的ApplicationListener
		2、将每个监听器添加到事件派发器中；
			getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
		3、派发之前步骤产生的事件；
```
<a name="Ffo5m"></a>
#### 5. 开始初始化
```java

11、finishBeanFactoryInitialization(beanFactory);初始化所有剩下的单实例bean；
	1、beanFactory.preInstantiateSingletons();初始化后剩下的单实例bean
		1）、获取容器中的所有Bean，依次进行初始化和创建对象
		2）、获取Bean的定义信息；RootBeanDefinition
		3）、Bean不是抽象的，是单实例的，不是懒加载；
			1）、判断是否是FactoryBean；是否是实现FactoryBean接口的Bean，调用getObject()获取对象；
			2）、不是工厂Bean。利用getBean(beanName);创建对象
				0、getBean(beanName)； ioc.getBean();
				1、doGetBean(name, null, null, false);
				2、先获取缓存中保存的单实例Bean。如果能获取到说明这个Bean之前被创建过（所有创建过的单实例Bean都会被缓存起来）
					从private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);获取的,依次从一二三级缓存中获取，从第三级缓存中获取的bean后，会加入到二级缓存中，并从第三级中删除
				3、缓存中获取不到，开始Bean的创建对象流程；
				4、标记当前bean已经被创建
				5、获取Bean的定义信息；
				6、【获取当前Bean依赖的其他Bean;如果有按照getBean()把依赖的Bean先创建出来；】
				7、启动单实例Bean的创建流程；
					1）、createBean(beanName, mbd, args);
					2）、Object bean = resolveBeforeInstantiation(beanName, mbdToUse);让BeanPostProcessor先拦截返回代理对象；
						【InstantiationAwareBeanPostProcessor】：提前执行；
						先触发：postProcessBeforeInstantiation()；
						如果有返回值：触发postProcessAfterInitialization()；
					3）、如果前面的InstantiationAwareBeanPostProcessor没有返回代理对象；调用4）
					
```
<a name="Tgyj1"></a>
#### 6. 实际创建Bean过程
```java
4）、Object beanInstance = doCreateBean(beanName, mbdToUse, args);创建Bean
	1）、【创建Bean实例】；createBeanInstance(beanName, mbd, args);
		利用工厂方法或者对象的构造器创建出Bean实例；
	2）、applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
        调用MergedBeanDefinitionPostProcessor的postProcessMergedBeanDefinition(mbd, beanType, beanName);
		addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
        这里会将实例化好的bean加入到三级缓存里,用于解决循环依赖问题
        接下来是初始化bean
    3）、【Bean属性赋值】populateBean(beanName, mbd, instanceWrapper);
                        赋值之前：
        1）、拿到InstantiationAwareBeanPostProcessor后置处理器；
                        postProcessAfterInstantiation()；
        2）、拿到InstantiationAwareBeanPostProcessor后置处理器；
                        postProcessPropertyValues()；
                        =====赋值之前：===
        3）、应用Bean属性的值；为属性利用setter方法等进行赋值；
                        applyPropertyValues(beanName, mbd, bw, pvs);
	4）、【Bean初始化】initializeBean(beanName, exposedObject, mbd);
        1）、【执行Aware接口方法】invokeAwareMethods(beanName, bean);执行xxxAware接口的方法
             BeanNameAware\BeanClassLoaderAware\BeanFactoryAware
        2）、【执行后置处理器初始化之前】applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
             BeanPostProcessor.postProcessBeforeInitialization（）;@PostConstruct这里执行
        3）、【执行初始化方法】invokeInitMethods(beanName, wrappedBean, mbd);
                  1）、是否是InitializingBean接口的实现；执行接口规定的初始化；
                  2）、是否自定义初始化方法；
        4）、【执行后置处理器初始化之后】applyBeanPostProcessorsAfterInitialization
                BeanPostProcessor.postProcessAfterInitialization()；
	 5）、注册Bean的销毁方法；
5）、将创建的Bean添加到缓存中singletonObjects；
ioc容器就是这些Map；很多的Map里面保存了单实例Bean，环境信息。。。。；
所有Bean都利用getBean创建完成以后；
检查所有的Bean是否是SmartInitializingSingleton接口的；如果是；就执行afterSingletonsInstantiated()；
```
<a name="ZCRXI"></a>
#### 7. 容器初始化完成，需要刷新
```java
12、finishRefresh();完成BeanFactory的初始化创建工作；IOC容器就创建完成；
		1）、initLifecycleProcessor();初始化和生命周期有关的后置处理器；LifecycleProcessor
			默认从容器中找是否有lifecycleProcessor的组件【LifecycleProcessor】；如果没有new DefaultLifecycleProcessor();
			加入到容器；
			
			写一个LifecycleProcessor的实现类，可以在BeanFactory
				void onRefresh();
				void onClose();	
		2）、	getLifecycleProcessor().onRefresh();
			拿到前面定义的生命周期处理器（BeanFactory）；回调onRefresh()；
		3）、publishEvent(new ContextRefreshedEvent(this));发布容器刷新完成事件；
		4）、liveBeansView.registerApplicationContext(this);
```
<a name="wCBgY"></a>
#### 8. 总结
```java
	
	======总结===========
	1）、Spring容器在启动的时候，先会保存所有注册进来的Bean的定义信息；
		1）、xml注册bean；<bean>
		2）、注解注册Bean；@Service、@Component、@Bean、xxx
	2）、Spring容器会合适的时机创建这些Bean
		1）、用到这个bean的时候；利用getBean创建bean；创建好以后保存在容器中；
		2）、统一创建剩下所有的bean的时候；finishBeanFactoryInitialization()；
	3）、后置处理器；BeanPostProcessor
		1）、每一个bean创建完成，都会使用各种后置处理器进行处理；来增强bean的功能；
			AutowiredAnnotationBeanPostProcessor:处理自动注入
			AnnotationAwareAspectJAutoProxyCreator:来做AOP功能；
			xxx....
			增强的功能注解：
			AsyncAnnotationBeanPostProcessor
			....
	4）、事件驱动模型；
		ApplicationListener；事件监听；
		ApplicationEventMulticaster；事件派发：
```

<a name="XGU09"></a>
## Spring AOP 流程
```java
 AOP：【动态代理】
 		指在程序运行期间动态的将某段代码切入到指定方法指定位置进行运行的编程方式；
 
 1、导入aop模块；Spring AOP：(spring-aspects)
 2、定义一个业务逻辑类（MathCalculator）；在业务逻辑运行的时候将日志进行打印（方法之前、方法运行结束、方法出现异常，xxx）
 3、定义一个日志切面类（LogAspects）：切面类里面的方法需要动态感知MathCalculator.div运行到哪里然后执行；
 		通知方法：
 			前置通知(@Before)：logStart：在目标方法(div)运行之前运行
 			后置通知(@After)：logEnd：在目标方法(div)运行结束之后运行（无论方法正常结束还是异常结束）
 			返回通知(@AfterReturning)：logReturn：在目标方法(div)正常返回之后运行
 			异常通知(@AfterThrowing)：logException：在目标方法(div)出现异常以后运行
 			环绕通知(@Around)：动态代理，手动推进目标方法运行（joinPoint.procced()）
 4、给切面类的目标方法标注何时何地运行（通知注解）；
 5、将切面类和业务逻辑类（目标方法所在类）都加入到容器中;
 6、必须告诉Spring哪个类是切面类(给切面类上加一个注解：@Aspect)
 [7]、给配置类中加 @EnableAspectJAutoProxy 【开启基于注解的aop模式】
 		在Spring中很多的 @EnableXXX;
 
 三步：
 	1）、将业务逻辑组件和切面类都加入到容器中；告诉Spring哪个是切面类（@Aspect）
 	2）、在切面类上的每一个通知方法上标注通知注解，告诉Spring何时何地运行（切入点表达式）
  3）、开启基于注解的aop模式；@EnableAspectJAutoProxy
```
```java
AOP原理：【看给容器中注册了什么组件，这个组件什么时候工作，这个组件的功能是什么？】
 		@EnableAspectJAutoProxy；
 1、@EnableAspectJAutoProxy是什么？
 		@Import(AspectJAutoProxyRegistrar.class)：给容器中导入AspectJAutoProxyRegistrar
 			利用AspectJAutoProxyRegistrar自定义给容器中注册bean；BeanDefinetion
 			internalAutoProxyCreator=AnnotationAwareAspectJAutoProxyCreator

 		给容器中注册一个AnnotationAwareAspectJAutoProxyCreator；

 2、 AnnotationAwareAspectJAutoProxyCreator：
 		AnnotationAwareAspectJAutoProxyCreator
 			->AspectJAwareAdvisorAutoProxyCreator
 				->AbstractAdvisorAutoProxyCreator
 					->AbstractAutoProxyCreator
 							implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware
 						关注后置处理器（在bean初始化完成前后做事情）、自动装配BeanFactory

 AbstractAutoProxyCreator.setBeanFactory()
 AbstractAutoProxyCreator.有后置处理器的逻辑；

 AbstractAdvisorAutoProxyCreator.setBeanFactory()-》initBeanFactory()

 AnnotationAwareAspectJAutoProxyCreator.initBeanFactory()
```
```java
流程：
 		1）、传入配置类，创建ioc容器
 		2）、注册配置类，调用refresh（）刷新容器；
 		3）、registerBeanPostProcessors(beanFactory);注册bean的后置处理器来方便拦截bean的创建；
 			1）、先获取ioc容器已经定义了的需要创建对象的所有BeanPostProcessor
 			2）、给容器中加别的BeanPostProcessor
 			3）、优先注册实现了PriorityOrdered接口的BeanPostProcessor；
 					先注册优先排序bpp是为了后续创建不优先的bpp，也会调用之前已经优先注册bpp的before
 			4）、再给容器中注册实现了Ordered接口的BeanPostProcessor；
 			5）、注册没实现优先级接口的BeanPostProcessor；
 			6）、注册BeanPostProcessor，实际上就是创建BeanPostProcessor对象，保存在容器中,这里也是走了创建bean的过程，也被bpp作用
 				创建internalAutoProxyCreator的BeanPostProcessor【AnnotationAwareAspectJAutoProxyCreator】
 				1）、创建Bean的实例
 				2）、populateBean；给bean的各种属性赋值
 				3）、initializeBean：初始化bean；
 						1）、invokeAwareMethods()：处理Aware接口的方法回调
 						2）、applyBeanPostProcessorsBeforeInitialization()：应用后置处理器的postProcessBeforeInitialization（）
 						3）、invokeInitMethods()；执行自定义的初始化方法
 						4）、applyBeanPostProcessorsAfterInitialization()；执行后置处理器的postProcessAfterInitialization（）；
 				4）、BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)创建成功；--》aspectJAdvisorsBuilder
 			7）、把BeanPostProcessor注册到BeanFactory中；
 				beanFactory.addBeanPostProcessor(postProcessor);
 =======以上是创建和注册AnnotationAwareAspectJAutoProxyCreator的过程========
```
```java
* 			AnnotationAwareAspectJAutoProxyCreator => InstantiationAwareBeanPostProcessor
 * 		4）、finishBeanFactoryInitialization(beanFactory);完成BeanFactory初始化工作；创建剩下的单实例bean
 * 			1）、遍历获取容器中所有的Bean，依次创建对象getBean(beanName);
 * 				getBean->doGetBean()->getSingleton()->
 * 			2）、创建bean
 * 				【AnnotationAwareAspectJAutoProxyCreator在所有bean创建之前会有一个拦截，InstantiationAwareBeanPostProcessor，会调用postProcessBeforeInstantiation()】
 * 				1）、先从缓存中获取当前bean，如果能获取到，说明bean是之前被创建过的，直接使用，否则再创建；
 * 					只要创建好的Bean都会被缓存起来
 * 				2）、createBean（）;创建bean；
 * 					AnnotationAwareAspectJAutoProxyCreator 会在任何bean创建之前先尝试返回bean的实例
 * 					【BeanPostProcessor是在Bean对象创建完成初始化前后调用的】
 * 					【InstantiationAwareBeanPostProcessor是在创建Bean实例之前先尝试用后置处理器返回对象的】
 * 					1）、resolveBeforeInstantiation(beanName, mbdToUse);解析BeforeInstantiation
 * 						希望后置处理器在此能返回一个代理对象；如果能返回代理对象就使用，如果不能就继续
 * 						1）、后置处理器先尝试返回对象；
 * 							bean = applyBeanPostProcessorsBeforeInstantiation（）：
 * 								拿到所有后置处理器，如果是InstantiationAwareBeanPostProcessor;
 * 								就执行postProcessBeforeInstantiation
 * 							if (bean != null) {
								bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
							}
 *
 * 					2）、doCreateBean(beanName, mbdToUse, args);真正的去创建一个bean实例；和3.6流程一样；
 * 					3）、
 *
 *
 * AnnotationAwareAspectJAutoProxyCreator【InstantiationAwareBeanPostProcessor】	的作用：
 * 1）、每一个bean创建之前，调用postProcessBeforeInstantiation()；
 * 		关心MathCalculator和LogAspect的创建
 * 		1）、判断当前bean是否在advisedBeans中（保存了所有需要增强bean）
 * 		2）、判断当前bean是否是基础类型的Advice、Pointcut、Advisor、AopInfrastructureBean，
 * 			或者是否是切面（@Aspect）
 * 		3）、是否需要跳过
 * 			1）、获取候选的增强器（切面里面的通知方法）【List<Advisor> candidateAdvisors】
 * 				每一个封装的通知方法的增强器是 InstantiationModelAwarePointcutAdvisor；
 * 				判断每一个增强器是否是 AspectJPointcutAdvisor 类型的；返回true
 * 			2）、永远返回false
 *
 * 2）、创建对象
 * postProcessAfterInitialization；
 * 		return wrapIfNecessary(bean, beanName, cacheKey);//包装如果需要的情况下
 * 		1）、获取当前bean的所有增强器（通知方法）  Object[]  specificInterceptors
 * 			1、找到候选的所有的增强器（找哪些通知方法是需要切入当前bean方法的）
 * 			2、获取到能在bean使用的增强器。
 * 			3、给增强器排序
 * 		2）、保存当前bean在advisedBeans中；
 * 		3）、如果当前bean需要增强，创建当前bean的代理对象；
 * 			1）、获取所有增强器（通知方法）
 * 			2）、保存到proxyFactory
 * 			3）、创建代理对象：Spring自动决定
 * 				JdkDynamicAopProxy(config);jdk动态代理；
 * 				ObjenesisCglibAopProxy(config);cglib的动态代理；
 * 		4）、给容器中返回当前组件使用cglib增强了的代理对象；
 * 		5）、以后容器中获取到的就是这个组件的代理对象，执行目标方法的时候，代理对象就会执行通知方法的流程；
 *
 *
 * 	3）、目标方法执行	；
 * 		容器中保存了组件的代理对象（cglib增强后的对象），这个对象里面保存了详细信息（比如增强器，目标对象，xxx）；
 * 		1）、CglibAopProxy.intercept();拦截目标方法的执行
 * 		2）、根据ProxyFactory对象获取将要执行的目标方法拦截器链；
 * 			List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
 * 			1）、List<Object> interceptorList保存所有拦截器 5
 * 				一个默认的ExposeInvocationInterceptor 和 4个增强器；
 * 			2）、遍历所有的增强器，将其转为Interceptor；
 * 				registry.getInterceptors(advisor);
 * 			3）、将增强器转为List<MethodInterceptor>；
 * 				如果是MethodInterceptor，直接加入到集合中
 * 				如果不是，使用AdvisorAdapter将增强器转为MethodInterceptor；
 * 				转换完成返回MethodInterceptor数组；
 *
 * 		3）、如果没有拦截器链，直接执行目标方法;
 * 			拦截器链（每一个通知方法又被包装为方法拦截器，利用MethodInterceptor机制）
 * 		4）、如果有拦截器链，把需要执行的目标对象，目标方法，
 * 			拦截器链等信息传入创建一个 CglibMethodInvocation 对象，
 * 			并调用 Object retVal =  mi.proceed();
 * 		5）、拦截器链的触发过程;
 * 			1)、如果没有拦截器执行执行目标方法，或者拦截器的索引和拦截器数组-1大小一样（指定到了最后一个拦截器）执行目标方法；
 * 			2)、链式获取每一个拦截器，拦截器执行invoke方法，每一个拦截器等待下一个拦截器执行完成返回以后再来执行；
 * 				拦截器链的机制，保证通知方法与目标方法的执行顺序；
 *
 */
```
```java
 	总结：
 		1）、  @EnableAspectJAutoProxy 开启AOP功能
 		2）、 @EnableAspectJAutoProxy 会给容器中注册一个组件 AnnotationAwareAspectJAutoProxyCreator
 		3）、AnnotationAwareAspectJAutoProxyCreator是一个后置处理器；
 		4）、容器的创建流程：
 			1）、registerBeanPostProcessors（）注册后置处理器；创建AnnotationAwareAspectJAutoProxyCreator对象
 			2）、finishBeanFactoryInitialization（）初始化剩下的单实例bean
 				1）、创建业务逻辑组件和切面组件
 				2）、AnnotationAwareAspectJAutoProxyCreator拦截组件的创建过程
 				3）、组件创建完之后，判断组件是否需要增强
 					是：切面的通知方法，包装成增强器（Advisor）;给业务逻辑组件创建一个代理对象（cglib）；
 		5）、执行目标方法：
 			1）、代理对象执行目标方法
 			2）、CglibAopProxy.intercept()；
 				1）、得到目标方法的拦截器链（增强器包装成拦截器MethodInterceptor）
 				2）、利用拦截器的链式机制，依次进入每一个拦截器进行执行；
 				3）、效果：
 					正常执行：前置通知-》目标方法-》后置通知-》返回通知
 					出现异常：前置通知-》目标方法-》后置通知-》异常通知
```
