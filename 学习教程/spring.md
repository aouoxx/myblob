



## Spring IOC



#### BeanDefinition



#### RootBeanDefinition

```java
public class RootBeanDefinition{
	// BeanDefinitionHolder 存储有Bean的名称,别名,BeanDefinition
	private BeanDefinitionHolder decoratedDefinition;
	// java反射包的接口,通过它可以查看Bean的注解信息
	private AnnotatedElement qualifiedElement;
	// 允许缓存
	boolean allowCaching = true;
	// 字面意思: 工厂方法是否唯一
	boolean isFactoryMethodUnique = false;
	// 封装了java.lang.reflect.Type 提供了泛型相关的操作
	volatile ResolvableType targetType;
	// 缓存class,表明RootBeanDefinition存储哪个类的信息
	@Nullable
	volatile Class<?> resolvedTargetType;
	
    // 缓存工厂方法的返回类型
	@Nullable
	volatile ResolvableType factoryMethodReturnType;

	// 缓存工厂方法
	@Nullable
	volatile Method factoryMethodToIntrospect;

	// 以下四个变量的锁
	final Object constructorArgumentLock = new Object();

	// 缓存已经解析的构造函数或工厂方法,Executable是Method,Constructor类型的父类
	@Nullable
	Executable resolvedConstructorOrFactoryMethod;

	// 表明构造函数参数是否已经解析完毕
	boolean constructorArgumentsResolved = false;

	// 缓存完全解析的构造函数的参数
	@Nullable
	Object[] resolvedConstructorArguments;

	// 缓存待解析的构造函数,即还没有找到对应的实例,可以理解为还没有注入依赖的形参
	@Nullable
	Object[] preparedConstructorArguments;

	// 下面两个变量的锁
	final Object postProcessingLock = new Object();

	// 表明是否被MergeBeanDefinitionPostProcessor处理过
	boolean postProcessed = false;

	// 在生成代理的时候会使用,表明是否已经生成代理
	@Nullable
	volatile Boolean beforeInstantiationResolved;

	@Nullable
	private Set<Member> externallyManagedConfigMembers;

    // InitializingBean的init回调函数名——afterPropertiesSet会记录在这里,以便进行生命周期回顾
	@Nullable
	private Set<String> externallyManagedInitMethods;
	// DisposableBean的destory回调函数名——destory会在这里记录,以便进行生命周期回调
	@Nullable
	private Set<String> externallyManagedDestroyMethods;

    
    
    RootBeanDefinition保存
        
    定义了id,别名和Bean的对应关系(BeanDefinitionHolder)
    Bean的注解(AnnotatedElement)  
    具体的工厂方法(Class类型),包括工厂方法的返回类型,工厂方法的Method对象
    构造函数,构造函数形参类型
    Bean的Class对象
```



#### MergedBeanDefinition

```java
BeanDefinition有一个MergedBeanDefinition的概念,对应的类还是RootBeanDefinition
主要作用是如果对应的BeanDefinion有父类,会将父类和子类的属性合并,并用子类的属性覆盖父类的属性。

> getBean(String beanName)实际上是通过MergedBeanDefinition来获取数据的
> spring专门其提供了一个生命周期回调定义接口“MergedBeanDefinitionPostProcessor”用于扩展


1) 根据原始BeanDefinition及其可能存在的双亲
```











```java
PropertyValue
	这是一个用来表示Bean属性的对象,其中定义了属性的名字和值等信息,如simpleService和SimpleDao属性
ProertyDescriptor
	这个是Bean属性的描述符,其中定义了该属性可能存在的setter和getter方法,以及所有Bean的Class对象

InjectionMetadata
	这个是注入元数据,包含了目标Bean的Class对象 和注入元素(InjectElement)集合

InjectionElement
	这个是注入元素,包含了注入元素的java.lang.reflect.Member的对象,以及一个PropertyDescriptor对象。
	就是对java.lang.reflect.Member的一个封装,用来执行最终的注入动作,
	它有两个子类,分别是AutowireFieldElement表示字段属性,AutowireMethodElement表示方法。
最终的目标就是将PropertyValue中的value值赋给InjectElement中的Member对象。

```

















## Spring 工具类

### BridgeMethodResolver

```java
什么是桥接方法
	桥接方法是JDK1.5引入泛型后,为了使Java的泛型方法生成的字节码和1.5版本前的字节码兼容,由编译器自动生成的方法。
	我们可以通过Method.isBridge()来判断一个方法是不是桥接方法,在字节码中桥接方法会被标记为ACC_BRIDGE和ACC_SYNCTHIC,
	ACC_BRIDGE 用于说明这个方法是不是由编译生成的桥接方法
	ACC_SYNTHETIC 说明这个方法是否由编辑器生成,并且不再源代码出现
	
什么时候会生成桥接方法
	一个子类在继承(或实现)一个父类(或接口)的泛型方法时,在子类中明确指定了泛型类型,那么在编辑时编译器会自动生成桥接方法
	当然还有其他情况也会生成桥接方法,这里只是列举了其中一种情况
	

https://blog.csdn.net/mhmyqn/article/details/47342577
```

***通过桥接方法获取实际方法***

```
我们在通过反射进行方法调用时,获取到桥接方法对应的实际方法。具体可以查看org.springframework.core.BridgeMethodResolver类的源码。
实际上是通过判断方法名,参数个数,以及泛型类型参数来获取的
```











## springboot

#### @ConfigutationProperties

```java
@ConfigurationProperties(prefix = "server", ignoreUnknownFields = true)
// ServerProperties 是一个配置属性对象，会以一个bean的形式注册到容器
public class ServerProperties {

	// Server HTTP port.
	private Integer port;

	// Network address to which the server should bind.
	private InetAddress address;
	//	...
}

当在一个bean组件上使用了@ConfigurationProperties注解时,指定的配置属性将会被设置到该bean。
这一注解背后的工作机制,主要是有框架的BeanPostProcessor ConfigurationPropertiesBindingPostProcessor来完成。

ConfigurationPropertiesBindingPostProcessor的作用是绑定PropertySources到@ConfigurationProperties注解的bean实例。
在实例化bean的时候,框架会使用它将@ConfigurationProperties注解中的指定前缀的外部配置项加载进行设置到bean相应的属性上。
这里的PropertySource能够从上下文中的Environment对象获取外部配置属性项


```





#### @Configuration

```java
Spring的工具类ConfigurationClassParser用于分析@Configuration注解的配置类,产生一组ConfigurationClass对象。

```



```java
ConfigurationClassParse 所在包org.springframework.context.annotation

这个工具类自身逻辑并不注册bean定义,它的主要任务是发现@Configuration注解的所有配置类并将这些配置类交给调用者(调用者会通过其他方式注册其中的bean定义)
而对于非@Configuration注解的其他bean定义,比如@Component注解的bean定义,该工具类使用另外一个工具ComponentScanAnnotationParser扫描和注册它们。

ComponentScanAnnotationParser该工具类@CompentScan, @CompentScan注解使用了ComponentScanAnnotationParser
ComponentScanAnnotationParser在扫描到bean定义时会直接将其注册到容器,而不是采用和ConfigutationClassParser类似的交由调用者处理。


一般情况,一个@Configuration注解的类只会产生一个ConfigurationClass对象,但是因为@Configuration注解的类可能会使用注解@Import引入其他配置类
也可能内部嵌套定义配置类,所以总的来看,ConfigurationClassParse分析一个@Configuration注解的类,可能产生任意多个ConfigurationClass对象

```





##  总结



