https://spring.io/
# Spring6
## AOT
jit 即时编译 
aot 提前编译 GraalVM实现

* 不用预热, 包体积小
* 无动态能力, 不能跨平台

现在正处于云原生，降本增效的时代，Java 相比于Go、Rust 等其他编程语言非常大的弊端就是启动编译和启动进程非常慢，这对于根据实时计算资源，弹性扩缩容的云原生技术相冲突，Spring6 借助AOT技术在运行时内存占用低，启动速度快，逐渐的来满足Java在云原生时代的需求，对于大规模使用Java 应用的商业公司可以考虑尽早调研使用JDK17,通过云原生技术为公司实现降本增效。


### Native Image
目前业界除了这种在JVM中进行AOT的方案，还有另外-种实现Java AOT的思路，那就是直接摒弃JVM,和
C/C++-样通过编译器直接将代码编译成机器代码，然后运行。这无疑是一种直接颠覆Java语言设计的思路，那就是GraalVM Native Image。它通过C语言实现了一个超微缩的运行时组件--Substrate VM,基本实现了JVM的各种特性，但足够轻量、可以被轻松内嵌，这就让ava语言和工程摆脱JVM的限制，能够真正意义上实现和C/C++一样的AOT编译。这一方案在经过长时间的优化和积累后，已经拥有非常不错的效果，基本上成为Oracle官方首推的Java AOT解决方案。
Native Image是一项创新技术，可将Java代码编译成独立的本机可执行文件或本机共享库。在构建本机可执行文件期间处理的Java字节码包括所有应用程序类、依赖项、第三方依赖库和任何所需的JDK类。生成的自包含本机可执行文件特定于不需要JVM的每个单独的操作系统和机器体系结构。















# Spring注解

## 1.IOC

### 1.组件注册

#### 1.@Configuration/@ComponentScan/@Filter/@Bean

```java
@Configuration
@ComponentScan
@ComponentScans
@Filter
@Bean
```

```java
//配置类==配置文件
@Configuration  //告诉Spring这是一个配置类
@ComponentScans(
    value = {
        @ComponentScan(value="com.atguigu",includeFilters = {
          /*@Filter(type=FilterType.ANNOTATION,classes={Controller.class}),
			@Filter(type=FilterType.ASSIGNABLE_TYPE,classes={BookService.class}),*/
            @Filter(type=FilterType.CUSTOM,classes={MyTypeFilter.class})
        },useDefaultFilters = false)//禁用默认过滤	
    }
)
//@ComponentScan  value:指定要扫描的包
//excludeFilters = Filter[] ：指定扫描的时候按照什么规则排除那些组件
//includeFilters = Filter[] ：指定扫描的时候只需要包含哪些组件
//FilterType.ANNOTATION：按照注解
//FilterType.ASSIGNABLE_TYPE：按照给定的类型；
//FilterType.ASPECTJ：使用ASPECTJ表达式
//FilterType.REGEX：使用正则指定
//FilterType.CUSTOM：使用自定义规则
public class MainConfig {

    //容器中注册一个Bean;类型为返回值的类型，id默认是用方法名作为id,也可以自定义id
    @Bean("person")
    public Person person01(){
        return new Person("lisi", 20);
    }

}
```

```java
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(config.class);
for (String beanDefinitionName : applicationContext.getBeanDefinitionNames()) {
    System.out.println(beanDefinitionName);
}
```

##### 1.自定义filer过滤规则

```java
public class MyTypeFilter implements TypeFilter {

	/**
	 * metadataReader：读取到的当前正在扫描的类的信息
	 * metadataReaderFactory:可以获取到其他任何类信息的
	 */
	@Override
	public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory)
			throws IOException {
		//获取当前类注解的信息
		AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
		//获取当前正在扫描的类的类信息
		ClassMetadata classMetadata = metadataReader.getClassMetadata();
		//获取当前类资源（类的路径）
		Resource resource = metadataReader.getResource();
		
		String className = classMetadata.getClassName();
		System.out.println("--->"+className);
		if(className.contains("er")){
			return true;
		}
		return false;
	}

}
```

#### 2.@Scope/@Lazy/@Conditional/@Import/FactoryBean

```java
@Scope//调整作用域
@Lazy//懒加载
@Conditional({Condition})//按照一定的条件进行判断，满足条件给容器中注册bean
@Import//快速给容器中导入一个组件
```

```java
//类中组件统一设置。满足当前条件，这个类中配置的所有bean注册才能生效；
@Conditional({WindowsCondition.class})
@Configuration
@Import({Color.class,Red.class,MyImportSelector.class,MyImportBeanDefinitionRegistrar.class})
//@Import导入组件，id默认是组件的全类名
public class MainConfig2 {
	
	//默认是单实例的
	/**
	 * ConfigurableBeanFactory#SCOPE_PROTOTYPE    
	 * @see ConfigurableBeanFactory#SCOPE_SINGLETON  
	 * @see org.springframework.web.context.WebApplicationContext#SCOPE_REQUEST  request
	 * @see org.springframework.web.context.WebApplicationContext#SCOPE_SESSION	 sesssion
	 * 
	 * prototype：多实例的：ioc容器启动并不会去调用方法创建对象放在容器中。
	 * 					每次获取的时候才会调用方法创建对象；
	 * singleton：单实例的（默认值）：ioc容器启动会调用方法创建对象放到ioc容器中。
	 * 			以后每次获取就是直接从容器（map.get()）中拿，
	 * request：同一次请求创建一个实例
	 * session：同一个session创建一个实例
	 * 
	 * 懒加载：
	 * 		单实例bean：默认在容器启动的时候创建对象；
	 * 		懒加载：容器启动不创建对象。第一次使用(获取)Bean创建对象，并初始化；
	 * 
	 */
//	@Scope("prototype")
	@Lazy
	@Bean("person")
	public Person person(){
		System.out.println("给容器中添加Person....");
		return new Person("张三", 25);
	}
	
	/**
	 * @Conditional({Condition}) ： 按照一定的条件进行判断，满足条件给容器中注册bean
	 * 
	 * 如果系统是windows，给容器中注册("bill")
	 * 如果是linux系统，给容器中注册("linus")
	 */
	@Bean("bill")
	public Person person01(){
		return new Person("Bill Gates",62);
	}
	@Conditional(LinuxCondition.class)
	@Bean("linus")
	public Person person02(){
		return new Person("linus", 48);
	}
	
	/**
	 * 给容器中注册组件；
	 * 1）、包扫描+组件标注注解（@Controller/@Service/@Repository/@Component）[自己写的类]
	 * 2）、@Bean[导入的第三方包里面的组件]
	 * 3）、@Import[快速给容器中导入一个组件]
	 * 		1）、@Import(要导入到容器中的组件)；容器中就会自动注册这个组件，id默认是全类名
	 * 		2）、ImportSelector:返回需要导入的组件的全类名数组；
	 * 		3）、ImportBeanDefinitionRegistrar:手动注册bean到容器中
	 * 4）、使用Spring提供的 FactoryBean（工厂Bean）;
	 * 		1）、默认获取到的是工厂bean调用getObject创建的对象
	 * 		2）、要获取工厂Bean本身，我们需要给id前面加一个&
	 * 										&colorFactoryBean
	 */
	@Bean
	public ColorFactoryBean colorFactoryBean(){
		return new ColorFactoryBean();
	}
}
```

##### 1.@Conditional自定义bean加载规则

```java
//判断是否linux系统
public class LinuxCondition implements Condition {

    /**
	 * ConditionContext：判断条件能使用的上下文（环境）
	 * AnnotatedTypeMetadata：注释信息
	 */
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        // TODO是否linux系统
        //1、能获取到ioc使用的beanfactory
        ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
        //2、获取类加载器
        ClassLoader classLoader = context.getClassLoader();
        //3、获取当前环境信息
        Environment environment = context.getEnvironment();
        //4、获取到bean定义的注册类
        BeanDefinitionRegistry registry = context.getRegistry();

        String property = environment.getProperty("os.name");

        //可以判断容器中的bean注册情况，也可以给容器中注册bean
        boolean definition = registry.containsBeanDefinition("person");
        if(property.contains("linux")){
            return true;
        }
        return false;
    }
}
public class WindowsCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        Environment environment = context.getEnvironment();
        String property = environment.getProperty("os.name");
        if(property.contains("Windows")){
            return true;
        }
        return false;
    }
}
```

##### 2.@Import-ImportSelector/ImportBeanDefinitionRegistrar

```java
//自定义逻辑返回需要导入的组件
public class MyImportSelector implements ImportSelector {
	//返回值，就是到导入到容器中的组件全类名
	//AnnotationMetadata:当前标注@Import注解的类的所有注解信息
	@Override
	public String[] selectImports(AnnotationMetadata importingClassMetadata) {
		//importingClassMetadata
		//方法不要返回null值
		return new String[]{"com.atguigu.bean.Blue","com.atguigu.bean.Yellow"};
	}
}
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
	/**
	 * AnnotationMetadata：当前类的注解信息
	 * BeanDefinitionRegistry:BeanDefinition注册类；
	 * 		把所有需要添加到容器中的bean；调用
	 * 		BeanDefinitionRegistry.registerBeanDefinition手工注册进来
	 */
	@Override
	public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
		boolean definition = registry.containsBeanDefinition("com.atguigu.bean.Red");
		boolean definition2 = registry.containsBeanDefinition("com.atguigu.bean.Blue");
		if(definition && definition2){
			//指定Bean定义信息；（Bean的类型，Bean。。。）
			RootBeanDefinition beanDefinition = new RootBeanDefinition(RainBow.class);
			//注册一个Bean，指定bean名
			registry.registerBeanDefinition("rainBow", beanDefinition);
		}
	}
}
```

#### 3.FactoryBean

```java
//创建一个Spring定义的FactoryBean
public class ColorFactoryBean implements FactoryBean<Color> {
	//返回一个Color对象，这个对象会添加到容器中
	@Override
	public Color getObject() throws Exception {
		System.out.println("ColorFactoryBean...getObject...");
		return new Color();
	}
	@Override
	public Class<?> getObjectType() {
		return Color.class;
	}
	//是单例？
	//true：这个bean是单实例，在容器中保存一份
	//false：多实例，每次获取都会创建一个新的bean；
	@Override
	public boolean isSingleton() {
		return false;
	}
}
```

```java
public void testImport(){
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);

    //工厂Bean获取的是调用getObject创建的对象
    Object bean2 = applicationContext.getBean("colorFactoryBean");
    Object bean3 = applicationContext.getBean("colorFactoryBean");
    System.out.println("bean的类型："+bean2.getClass());
    System.out.println(bean2 == bean3);

    //1）、默认获取到的是工厂bean调用getObject创建的对象
	//2）、要获取工厂Bean本身，我们需要给id前面加一个&
    Object bean4 = applicationContext.getBean("&colorFactoryBean");
    System.out.println(bean4.getClass());
}
```

### 2.Bean生命周期

```java
/**
 * bean的生命周期：
 * 		bean创建---初始化----销毁的过程
 * 容器管理bean的生命周期；
 * 我们可以自定义初始化和销毁方法；容器在bean进行到当前生命周期的时候来调用我们自定义的初始化和销毁方法
 * 
 * 构造（对象创建）
 * 		单实例：在容器启动的时候创建对象
 * 		多实例：在每次获取的时候创建对象\
 * 
 * BeanPostProcessor.postProcessBeforeInitialization
 * 初始化：
 * 		对象创建完成，并赋值好，调用初始化方法。。。
 * BeanPostProcessor.postProcessAfterInitialization
 * 销毁：
 * 		单实例：容器关闭的时候
 * 		多实例：容器不会管理这个bean；容器不会调用销毁方法；
 * 
 * 
 * 遍历得到容器中所有的BeanPostProcessor；挨个执行beforeInitialization，
 * 一但返回null，跳出for循环，不会执行后面的BeanPostProcessor.postProcessorsBeforeInitialization
 * 
 * BeanPostProcessor原理
 * populateBean(beanName, mbd, instanceWrapper);给bean进行属性赋值
 * initializeBean
 * {
 * applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
 * invokeInitMethods(beanName, wrappedBean, mbd);执行自定义初始化
 * applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
 *}
 * 
 * 
 * 
 * 1）、指定初始化和销毁方法；
 * 		通过@Bean指定init-method和destroy-method；
 * 2）、通过让Bean实现InitializingBean（定义初始化逻辑），
 * 				DisposableBean（定义销毁逻辑）;
 * 3）、可以使用JSR250；
 * 		@PostConstruct：在bean创建完成并且属性赋值完成；来执行初始化方法
 * 		@PreDestroy：在容器销毁bean之前通知我们进行清理工作
 * 4）、BeanPostProcessor【interface】：bean的后置处理器；
 * 		在bean初始化前后进行一些处理工作；
 * 		postProcessBeforeInitialization:在初始化之前工作
 * 		postProcessAfterInitialization:在初始化之后工作
 * 
 * Spring底层对 BeanPostProcessor 的使用；
 * 		bean赋值，注入其他组件，@Autowired，生命周期注解功能，@Async,xxx BeanPostProcessor;
 * 
 * @author lfy
 *
 */
@ComponentScan("com.atguigu.bean")
@Configuration
public class MainConfigOfLifeCycle {
	
	//@Scope("prototype")
	@Bean(initMethod="init",destroyMethod="detory")
	public Car car(){
		return new Car();
	}

}
```

#### 1.@Bean指定init/detory

```java
@Component
public class Car {
	public Car(){
		System.out.println("car constructor...");
	}
	public void init(){
		System.out.println("car ... init...");
	}
	public void detory(){
		System.out.println("car ... detory...");
	}
}
```

#### 2.Bean实现InitializingBean/DisposableBean

```java
@Component
public class Cat implements InitializingBean,DisposableBean {
	public Cat(){
		System.out.println("cat constructor...");
	}
	@Override
	public void destroy() throws Exception {
		System.out.println("cat...destroy...");
	}
	@Override
	public void afterPropertiesSet() throws Exception {
		System.out.println("cat...afterPropertiesSet...");
	}
}
```

#### 3.@PostConstruct/@PreDestroy

```java
@Component
public class Dog implements ApplicationContextAware {
	//@Autowired
	private ApplicationContext applicationContext;
	public Dog(){
		System.out.println("dog constructor...");
	}
	
	//对象创建并赋值之后调用
	@PostConstruct
	public void init(){
		System.out.println("Dog....@PostConstruct...");
	}
	//容器移除对象之前
	@PreDestroy
	public void detory(){
		System.out.println("Dog....@PreDestroy...");
	}
	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		this.applicationContext = applicationContext;
	}
}
```

#### 4.BeanPostProcessor

```java
/**
 * 后置处理器：初始化前后进行处理工作
 * 将后置处理器加入到容器中
 */
@Component
public class MyBeanPostProcessor implements BeanPostProcessor {
	@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		System.out.println("postProcessBeforeInitialization..."+beanName+"=>"+bean);
		return bean;
	}
	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		System.out.println("postProcessAfterInitialization..."+beanName+"=>"+bean);
		return bean;
	}
}
```

### 3.属性赋值@Value

```java
//使用@PropertySource读取外部配置文件中的k/v保存到运行的环境变量中;加载完外部的配置文件以后使用${}取出配置文件的值
@PropertySource(value={"classpath:/person.properties"})
@Configuration
public class MainConfigOfPropertyValues {
	@Bean
	public Person person(){
		return new Person();
	}
}

public class Person {	
	//使用@Value赋值；
	//1、基本数值
	//2、可以写SpEL； #{}
	//3、可以写${}；取出配置文件【properties】中的值（在运行环境变量里面的值）
	@Value("张三")
	private String name;
	@Value("#{20-2}")
	private Integer age;
	@Value("${person.nickName}")
	private String nickName;
```

```java
AnnotationConfigApplicationContext applicationContext = 
    new AnnotationConfigApplicationContext(MainConfigOfPropertyValues.class);

ConfigurableEnvironment environment = applicationContext.getEnvironment();
String property = environment.getProperty("person.nickName");
System.out.println(property);
```

### 4.自动装配

```java
/**
 * 自动装配;
 * 		Spring利用依赖注入（DI），完成对IOC容器中中各个组件的依赖关系赋值；
 * 
 * 1）、@Autowired：自动注入：
 * 		1）、默认优先按照类型去容器中找对应的组件:applicationContext.getBean(BookDao.class);找到就赋值
 * 		2）、如果找到多个相同类型的组件，再将属性的名称作为组件的id去容器中查找
 * 							applicationContext.getBean("bookDao")
 * 		3）、@Qualifier("bookDao")：使用@Qualifier指定需要装配的组件的id，而不是使用属性名
 * 		4）、自动装配默认一定要将属性赋值好，没有就会报错；
 * 			可以使用@Autowired(required=false);
 * 		5）、@Primary：让Spring进行自动装配的时候，默认使用首选的bean；
 * 				也可以继续使用@Qualifier指定需要装配的bean的名字
 * 		BookService{
 * 			@Autowired
 * 			BookDao  bookDao;
 * 		}
 * 
 * 2）、Spring还支持使用@Resource(JSR250)和@Inject(JSR330)[java规范的注解]
 * 		@Resource:
 * 			可以和@Autowired一样实现自动装配功能；默认是按照组件名称进行装配的；
 * 			没有能支持@Primary功能没有支持@Autowired（reqiured=false）;
 * 		@Inject:
 * 			需要导入javax.inject的包，和Autowired的功能一样。没有required=false的功能；
 *  @Autowired:Spring定义的； @Resource、@Inject都是java规范
 * 	
 * AutowiredAnnotationBeanPostProcessor:解析完成自动装配功能；		
 * 
 * 3）、 @Autowired:构造器，参数，方法，属性；都是从容器中获取参数组件的值
 * 		1）、[标注在方法位置]：@Bean+方法参数；参数从容器中获取;默认不写@Autowired效果是一样的；都能自动装配
 * 		2）、[标在构造器上]：如果组件只有一个有参构造器，这个有参构造器的@Autowired可以省略，参数位置的组件还是可以自动从容器中获取
 * 		3）、放在参数位置：
 * 
 * 4）、自定义组件想要使用Spring容器底层的一些组件（ApplicationContext，BeanFactory，xxx）；
 * 		自定义组件实现xxxAware；在创建对象的时候，会调用接口规定的方法注入相关组件；Aware；
 * 		把Spring底层一些组件注入到自定义的Bean中；
 * 		xxxAware：功能使用xxxProcessor；
 * 			ApplicationContextAware==》ApplicationContextAwareProcessor；
 */
@Configuration
@ComponentScan({"com.atguigu.service","com.atguigu.dao",
	"com.atguigu.controller","com.atguigu.bean"})
public class MainConifgOfAutowired {
	@Primary
	@Bean("bookDao2")
	public BookDao bookDao(){
		BookDao bookDao = new BookDao();
		bookDao.setLable("2");
		return bookDao;
	}
	/**
	 * @Bean标注的方法创建对象的时候，方法参数的值从容器中获取
	 * @param car
	 * @return
	 */
	@Bean
	public Color color(Car car){
		Color color = new Color();
		color.setCar(car);
		return color;
	}
}
```

```java
//默认加在ioc容器中的组件，容器启动会调用无参构造器创建对象，再进行初始化赋值等操作
@Component
public class Boss {
	private Car car;
	//构造器要用的组件，都是从容器中获取
    //@Autowired
	public Boss(@Autowired Car car){
		this.car = car;
		System.out.println("Boss...有参构造器");
	}
	public Car getCar() {
		return car;
	}
	//@Autowired 
	//标注在方法，Spring容器创建当前对象，就会调用方法，完成赋值；
	//方法使用的参数，自定义类型的值从ioc容器中获取
	public void setCar(Car car) {
		this.car = car;
	}
}
```

#### Aware

```java
@Component
public class Red implements ApplicationContextAware,BeanNameAware,EmbeddedValueResolverAware {
	private ApplicationContext applicationContext;
	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		System.out.println("传入的ioc："+applicationContext);
		this.applicationContext = applicationContext;
	}

	@Override
	public void setBeanName(String name) {
		System.out.println("当前bean的名字："+name);
	}

	@Override
	public void setEmbeddedValueResolver(StringValueResolver resolver) {
		String resolveStringValue = resolver.resolveStringValue("你好 ${os.name} 我是 #{20*18}");
		System.out.println("解析的字符串："+resolveStringValue);
	}
}

```

#### @Profile

```java
/**
 * Profile：
 * 		Spring为我们提供的可以根据当前环境，动态的激活和切换一系列组件的功能；
 * 
 * 开发环境、测试环境、生产环境；
 * 数据源：(/A)(/B)(/C)；
 * 
 * 
 * @Profile：指定组件在哪个环境的情况下才能被注册到容器中，不指定，任何环境下都能注册这个组件
 * 
 * 1）、加了环境标识的bean，只有这个环境被激活的时候才能注册到容器中。默认是default环境
 * 2）、写在配置类上，只有是指定的环境的时候，整个配置类里面的所有配置才能开始生效
 * 3）、没有标注环境标识的bean在，任何环境下都是加载的；
 */
@PropertySource("classpath:/dbconfig.properties")
@Configuration
public class MainConfigOfProfile implements EmbeddedValueResolverAware{
	@Value("${db.user}")
	private String user;
	private StringValueResolver valueResolver;
	private String  driverClass;
	
	@Bean
	public Yellow yellow(){
		return new Yellow();
	}
	@Profile("test")
	@Bean("testDataSource")
	public DataSource dataSourceTest(@Value("${db.password}")String pwd) throws Exception{
		ComboPooledDataSource dataSource = new ComboPooledDataSource();
		dataSource.setUser(user);
		dataSource.setPassword(pwd);
		dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
		dataSource.setDriverClass(driverClass);
		return dataSource;
	}
	@Profile("dev")
	@Bean("devDataSource")
	public DataSource dataSourceDev(@Value("${db.password}")String pwd) throws Exception{
		ComboPooledDataSource dataSource = new ComboPooledDataSource();
		dataSource.setUser(user);
		dataSource.setPassword(pwd);
		dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/ssm_crud");
		dataSource.setDriverClass(driverClass);
		return dataSource;
	}
	@Profile("prod")
	@Bean("prodDataSource")
	public DataSource dataSourceProd(@Value("${db.password}")String pwd) throws Exception{
		ComboPooledDataSource dataSource = new ComboPooledDataSource();
		dataSource.setUser(user);
		dataSource.setPassword(pwd);
		dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/scw_0515");
		dataSource.setDriverClass(driverClass);
		return dataSource;
	}
	@Override
	public void setEmbeddedValueResolver(StringValueResolver resolver) {
		this.valueResolver = resolver;
		driverClass = valueResolver.resolveStringValue("${db.driverClass}");
	}
}
```

```java
//1、使用命令行动态参数: 在虚拟机参数位置加载 -Dspring.profiles.active=test
//2、代码的方式激活某种环境；
@Test
public void test01(){
    AnnotationConfigApplicationContext applicationContext = 
        new AnnotationConfigApplicationContext();
    //1、创建一个applicationContext
    //2、设置需要激活的环境
    applicationContext.getEnvironment().setActiveProfiles("dev","test");
    //3、注册主配置类
    applicationContext.register(MainConfigOfProfile.class);
    //4、启动刷新容器
    applicationContext.refresh();


    String[] namesForType = applicationContext.getBeanNamesForType(DataSource.class);
    for (String string : namesForType) {
        System.out.println(string);
    }

    Yellow bean = applicationContext.getBean(Yellow.class);
    System.out.println(bean);
    applicationContext.close();
}
```

## 2.AOP

### AOP：【动态代理】

* 指在程序运行期间动态的将某段代码切入到指定方法指定位置进行运行的编程方式



1. 导入aop模块；Spring AOP：(spring-aspects)

2. 定义一个业务逻辑类（MathCalculator）；在业务逻辑运行的时候将日志进行打印（方法之前、方法运行结束、方法出现异常，xxx）

3. 定义一个日志切面类（LogAspects）：切面类里面的方法需要动态感知MathCalculator.div运行到哪里然后执行；

   通知方法：

   * 前置通知(@Before)：logStart：在目标方法(div)运行之前运行
   * 后置通知(@After)：logEnd：在目标方法(div)运行结束之后运行（无论方法正常结束还是异常结束）
   * 返回通知(@AfterReturning)：logReturn：在目标方法(div)正常返回之后运行
   * 异常通知(@AfterThrowing)：logException：在目标方法(div)出现异常以后运行
   * 环绕通知(@Around)：动态代理，手动推进目标方法运行（joinPoint.procced()）

4. 给切面类的目标方法标注何时何地运行（通知注解）；
5. 将切面类和业务逻辑类（目标方法所在类）都加入到容器中;
6. 必须告诉Spring哪个类是切面类(给切面类上加一个注解：@Aspect)
7. 给配置类中加 @EnableAspectJAutoProxy 【开启基于注解的aop模式】在Spring中很多的 @EnableXXX;



三步：

 * 	将业务逻辑组件和切面类都加入到容器中；告诉Spring哪个是切面类（@Aspect）
 * 	在切面类上的每一个通知方法上标注通知注解，告诉Spring何时何地运行（切入点表达式）
 *  开启基于注解的aop模式；@EnableAspectJAutoProxy

```java
@EnableAspectJAutoProxy
@Configuration
public class MainConfigOfAOP {
	//业务逻辑类加入容器中
	@Bean
	public MathCalculator calculator(){
		return new MathCalculator();
	}
	//切面类加入到容器中
	@Bean
	public LogAspects logAspects(){
		return new LogAspects();
	}
}
```

```java
public class MathCalculator {
	public int div(int i,int j){
		System.out.println("MathCalculator...div...");
		return i/j;	
	}
}
```

```java
/**
 * 切面类
 * @Aspect： 告诉Spring当前类是一个切面类
 */
@Aspect
public class LogAspects {

    //抽取公共的切入点表达式
    //1、本类引用
    //2、其他的切面引用 @After("com.atguigu.aop.LogAspects.pointCut()")
    @Pointcut("execution(public int com.atguigu.aop.MathCalculator.*(..))")
    public void pointCut(){};

    //@Before在目标方法之前切入；切入点表达式（指定在哪个方法切入）
    @Before("pointCut()")
    public void logStart(JoinPoint joinPoint){
        Object[] args = joinPoint.getArgs();
        System.out.println(""+joinPoint.getSignature().getName()+"运行。。。@Before:参数列表是：{"+Arrays.asList(args)+"}");
    }
    @After("com.atguigu.aop.LogAspects.pointCut()")
    public void logEnd(JoinPoint joinPoint){
        System.out.println(""+joinPoint.getSignature().getName()+"结束。。。@After");
    }
    //JoinPoint一定要出现在参数表的第一位
    @AfterReturning(value="pointCut()",returning="result")
    public void logReturn(JoinPoint joinPoint,Object result){
        System.out.println(""+joinPoint.getSignature().getName()+"正常返回。。。@AfterReturning:运行结果：{"+result+"}");
    }
    @AfterThrowing(value="pointCut()",throwing="exception")
    public void logException(JoinPoint joinPoint,Exception exception){
        System.out.println(""+joinPoint.getSignature().getName()+"异常。。。异常信息：{"+exception+"}");
    }
}
```

###  AOP原理：

【看给容器中注册了什么组件，这个组件什么时候工作，这个组件的功能是什么？】
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

AbstractAdvisorAutoProxyCreator.setBeanFactory()   -》 initBeanFactory()

AnnotationAwareAspectJAutoProxyCreator.initBeanFactory()

#### 流程：

##### 1. 传入配置类，创建ioc容器

##### 2.注册配置类，调用refresh（）刷新容器；

##### 3.registerBeanPostProcessors(beanFactory);注册bean的后置处理器来方便拦截bean的创建；

1. 先获取ioc容器已经定义了的需要创建对象的所有BeanPostProcessor

2. 给容器中加别的BeanPostProcessor

3. 优先注册实现了PriorityOrdered接口的BeanPostProcessor；

4. 再给容器中注册实现了Ordered接口的BeanPostProcessor；

5. 注册没实现优先级接口的BeanPostProcessor；

6. 注册BeanPostProcessor，实际上就是创建BeanPostProcessor对象，保存在容器中；
        创建internalAutoProxyCreator的BeanPostProcessor【AnnotationAwareAspectJAutoProxyCreator】

   1. 创建Bean的实例

   2. populateBean；给bean的各种属性赋值

   3. initializeBean：初始化bean；

      1. invokeAwareMethods()：处理Aware接口的方法回调

      2. applyBeanPostProcessorsBeforeInitialization()：

         应用后置处理器的postProcessBeforeInitialization（）

      3. invokeInitMethods()；执行自定义的初始化方法

      4. applyBeanPostProcessorsAfterInitialization()；
      
         执行后置处理器的postProcessAfterInitialization（）；
   
   4. BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)
   
      创建成功；--》aspectJAdvisorsBuilder
   
7. 把BeanPostProcessor注册到BeanFactory中；
                                     beanFactory.addBeanPostProcessor(postProcessor);

====以上是创建和注册AnnotationAwareAspectJAutoProxyCreator的过程

​			AnnotationAwareAspectJAutoProxyCreator => InstantiationAwareBeanPostProcessor

##### 4.finishBeanFactoryInitialization(beanFactory);完成BeanFactory初始化工作；创建剩下的单实例bean

1. 遍历获取容器中所有的Bean，依次创建对象getBean(beanName);
                           getBean->doGetBean()->getSingleton()->

2. 创建bean
   【AnnotationAwareAspectJAutoProxyCreator在所有bean创建之前会有一个拦截

   ​						InstantiationAwareBeanPostProcessor，会调用postProcessBeforeInstantiation()】

   1. 先从缓存中获取当前bean，如果能获取到，说明bean是之前被创建过的，直接使用，否则再创建；
                 只要创建好的Bean都会被缓存起来
   2. createBean（）;创建bean；
                 AnnotationAwareAspectJAutoProxyCreator 会在任何bean创建之前先尝试返回bean的实例
                 【BeanPostProcessor是在Bean对象创建完成初始化前后调用的】
                 【InstantiationAwareBeanPostProcessor是在创建Bean实例之前先尝试用后置处理器返回对象的】
      1. resolveBeforeInstantiation(beanName, mbdToUse);解析BeforeInstantiation
         希望后置处理器在此能返回一个代理对象；如果能返回代理对象就使用，如果不能就继续
         1. 后置处理器先尝试返回对象；
                         ```java
                         bean = applyBeanPostProcessorsBeforeInstantiation（）：
                             //拿到所有后置处理器，如果是InstantiationAwareBeanPostProcessor;
                           	//就执行postProcessBeforeInstantiation
                  if (bean != null) {
                      bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
                  }
                  
              ```
              
              ```
         
      2. doCreateBean(beanName, mbdToUse, args);真正的去创建一个bean实例；和3.6流程一样；



AnnotationAwareAspectJAutoProxyCreator【InstantiationAwareBeanPostProcessor】    的作用：

1. 每一个bean创建之前，调用postProcessBeforeInstantiation()；
   ​      关心MathCalculator和LogAspect的创建
   1. 判断当前bean是否在advisedBeans中（保存了所有需要增强bean）
   2. 判断当前bean是否是基础类型的Advice、Pointcut、Advisor、AopInfrastructureBean，
      ​         或者是否是切面（@Aspect）
   3. 是否需要跳过
      1. 获取候选的增强器（切面里面的通知方法）【List<Advisor> candidateAdvisors】
         ​            每一个封装的通知方法的增强器是 InstantiationModelAwarePointcutAdvisor；
         ​            判断每一个增强器是否是 AspectJPointcutAdvisor 类型的；返回true
      2. 永远返回false

2. 创建对象  postProcessAfterInitialization；
         return wrapIfNecessary(bean, beanName, cacheKey);//包装如果需要的情况下

   1. 获取当前bean的所有增强器（通知方法）  Object[]  specificInterceptors

      1. 找到候选的所有的增强器（找哪些通知方法是需要切入当前bean方法的）

      2. 获取到能在bean使用的增强器。

      3. 给增强器排序

   2. 保存当前bean在advisedBeans中；
3. 如果当前bean需要增强，创建当前bean的代理对象；
      1. 获取所有增强器（通知方法）
   2. 保存到proxyFactory
      3. 创建代理对象：Spring自动决定
                    JdkDynamicAopProxy(config);jdk动态代理；
                    ObjenesisCglibAopProxy(config);cglib的动态代理；
      4. 给容器中返回当前组件使用cglib增强了的代理对象；
      5. 以后容器中获取到的就是这个组件的代理对象，执行目标方法的时候，代理对象就会执行通知方法的流程；     

3. 目标方法执行  ；
   容器中保存了组件的代理对象（cglib增强后的对象），这个对象里面保存了详细信息（比如增强器，目标对象，xxx）；

   1. CglibAopProxy.intercept();拦截目标方法的执行

   2. 根据ProxyFactory对象获取将要执行的目标方法拦截器链；

      ```java
      List<Object> chain = 
          this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
      ```

      1. List<Object> interceptorList保存所有拦截器 5
                     一个默认的ExposeInvocationInterceptor 和 4个增强器；
      2. 遍历所有的增强器，将其转为Interceptor；
                     registry.getInterceptors(advisor);
      3. 将增强器转为List<MethodInterceptor>；
                     如果是MethodInterceptor，直接加入到集合中
                     如果不是，使用AdvisorAdapter将增强器转为MethodInterceptor；
                     转换完成返回MethodInterceptor数组；

   3. 如果没有拦截器链，直接执行目标方法;
                           拦截器链（每一个通知方法又被包装为方法拦截器，利用MethodInterceptor机制）

   4. 如果有拦截器链，把需要执行的目标对象，目标方法，
           拦截器链等信息传入创建一个 CglibMethodInvocation 对象，
           并调用 Object retVal =  mi.proceed();

   5. 拦截器链的触发过程;

      1. 如果没有拦截器执行执行目标方法，或者拦截器的索引和拦截器数组-1大小一样

         （指定到了最后一个拦截器）执行目标方法；

      2. 链式获取每一个拦截器，拦截器执行invoke方法，

         每一个拦截器等待下一个拦截器执行完成返回以后再来执行；
         拦截器链的机制，保证通知方法与目标方法的执行顺序；

   

#### 总结：

1.  @EnableAspectJAutoProxy 开启AOP功能

2. @EnableAspectJAutoProxy 会给容器中注册一个组件 AnnotationAwareAspectJAutoProxyCreator

3. AnnotationAwareAspectJAutoProxyCreator是一个后置处理器；

4. 容器的创建流程：

   1. registerBeanPostProcessors（）注册后置处理器；创建AnnotationAwareAspectJAutoProxyCreator对象

   2. finishBeanFactoryInitialization（）初始化剩下的单实例bean

      1. 创建业务逻辑组件和切面组件

      2. AnnotationAwareAspectJAutoProxyCreator拦截组件的创建过程

      3. 组件创建完之后，判断组件是否需要增强
                        是：切面的通知方法，包装成增强器（Advisor）;给业务逻辑组件创建一个代理对象（cglib）；

5. 执行目标方法：
   1. 代理对象执行目标方法
   2. CglibAopProxy.intercept()；
      1. 得到目标方法的拦截器链（增强器包装成拦截器MethodInterceptor）
      2. 利用拦截器的链式机制，依次进入每一个拦截器进行执行；
      3. 效果：
                        正常执行：前置通知-》目标方法-》后置通知-》返回通知
                        出现异常：前置通知-》目标方法-》后置通知-》异常通知



## 3.声明式事务

   环境搭建：

1、导入相关依赖
      数据源、数据库驱动、Spring-jdbc模块
2、配置数据源、JdbcTemplate（Spring提供的简化数据库操作的工具）操作数据
3、给方法上标注 @Transactional 表示当前方法是一个事务方法；
4、 @EnableTransactionManagement 开启基于注解的事务管理功能；
      @EnableXXX
5、配置事务管理器来控制事务;
                      @Bean
                  public PlatformTransactionManager transactionManager()

```java
@EnableTransactionManagement
@ComponentScan("com.atguigu.tx")
@Configuration
public class TxConfig {
	//数据源
	@Bean
	public DataSource dataSource() throws Exception{
		ComboPooledDataSource dataSource = new ComboPooledDataSource();
		dataSource.setUser("root");
		dataSource.setPassword("123456");
		dataSource.setDriverClass("com.mysql.jdbc.Driver");
		dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
		return dataSource;
	}
	@Bean
	public JdbcTemplate jdbcTemplate() throws Exception{
		//Spring对@Configuration类会特殊处理；给容器中加组件的方法，多次调用都只是从容器中找组件
		JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource());
		return jdbcTemplate;
	}
	//注册事务管理器在容器中
	@Bean
	public PlatformTransactionManager transactionManager() throws Exception{
		return new DataSourceTransactionManager(dataSource());
	}
}
```

### 原理

1. @EnableTransactionManagement
            利用TransactionManagementConfigurationSelector给容器中会导入组件
            导入两个组件
            AutoProxyRegistrar
            ProxyTransactionManagementConfiguration
2. AutoProxyRegistrar：
            给容器中注册一个 InfrastructureAdvisorAutoProxyCreator 组件；
            InfrastructureAdvisorAutoProxyCreator：？
            利用后置处理器机制在对象创建以后，包装对象，返回一个代理对象（增强器），代理对象执行方法利用拦截器链进行调用；

3. ProxyTransactionManagementConfiguration 做了什么？
   1. 给容器中注册事务增强器；
      1. 事务增强器要用事务注解的信息，AnnotationTransactionAttributeSource解析事务注解
      2. 事务拦截器：
              TransactionInterceptor；保存了事务属性信息，事务管理器；
              他是一个 MethodInterceptor；
              在目标方法执行的时候；
                       执行拦截器链；
                       事务拦截器：
                            1. 先获取事务相关的属性
                                               2. 再获取PlatformTransactionManager，如果事先没有添加指定任何transactionmanger
                                 最终会从容器中按照类型获取一个PlatformTransactionManager；
                                               3. 执行目标方法
                                 如果异常，获取到事务管理器，利用事务管理回滚操作；
                                 如果正常，利用事务管理器，提交事务



## 扩展原理

BeanPostProcessor：bean后置处理器，bean创建对象初始化前后进行拦截工作的



BeanFactoryPostProcessor：beanFactory的后置处理器；

​		在BeanFactory标准初始化之后调用，来定制和修改BeanFactory的内容；

​		所有的bean定义已经保存加载到beanFactory，但是bean的实例还未创建

### BeanFactoryPostProcessor

1. ioc容器创建对象

2. invokeBeanFactoryPostProcessors(beanFactory);

   如何找到所有的BeanFactoryPostProcessor并执行他们的方法；

   1. 直接在BeanFactory中找到所有类型是BeanFactoryPostProcessor的组件，并执行他们的方法
   2. 在初始化创建其他组件前面执行

```java
@Component
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
	@Override
	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
		System.out.println("MyBeanFactoryPostProcessor...postProcessBeanFactory...");
		int count = beanFactory.getBeanDefinitionCount();
		String[] names = beanFactory.getBeanDefinitionNames();
		System.out.println("当前BeanFactory中有"+count+" 个Bean");
		System.out.println(Arrays.asList(names));
	}
}
```

```java
public abstract class AbstractApplicationContext{
public void refresh() throws BeansException, IllegalStateException {
// Allows post-processing of the bean factory in context subclasses.
postProcessBeanFactory(beanFactory);//允许在上下文子类中对 bean 工厂进行后处理。

// Invoke factory processors registered as beans in the context.
invokeBeanFactoryPostProcessors(beanFactory);//调用在上下文中注册为 bean 的工厂处理器。

// Register bean processors that intercept bean creation.
registerBeanPostProcessors(beanFactory);//注册拦截 bean 创建的 bean 处理器。

// Initialize message source for this context.
initMessageSource();//初始化此上下文的消息源。

// Initialize event multicaster for this context.
initApplicationEventMulticaster();//为此上下文初始化事件多播器。

// Initialize other special beans in specific context subclasses.
onRefresh();//初始化特定上下文子类中的其他特殊 bean。

// Check for listener beans and register them.
registerListeners();//检查侦听器 bean 并注册它们。

// Instantiate all remaining (non-lazy-init) singletons.
finishBeanFactoryInitialization(beanFactory);//实例化所有剩余的（非延迟初始化）单例。

// Last step: publish corresponding event.
finishRefresh();//最后一步：发布相应的事件。
```

### BeanDefinitionRegistryPostProcessor 

​								extends BeanFactoryPostProcessor

​		postProcessBeanDefinitionRegistry();

​		在所有bean定义信息将要被加载，bean实例还未创建的；

​		优先于BeanFactoryPostProcessor执行；

​		利用BeanDefinitionRegistryPostProcessor给容器中再额外添加一些组件；

```java
@Component
public class MyBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor{

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        System.out.println("MyBeanDefinitionRegistryPostProcessor...bean的数量："+beanFactory.getBeanDefinitionCount());
    }

    //BeanDefinitionRegistry Bean定义信息的保存中心，以后BeanFactory就是按照BeanDefinitionRegistry里面保存的每一个bean定义信息创建bean实例；
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        System.out.println("postProcessBeanDefinitionRegistry...bean的数量："+registry.getBeanDefinitionCount());
        //RootBeanDefinition beanDefinition = new RootBeanDefinition(Blue.class);
        AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.rootBeanDefinition(Blue.class).getBeanDefinition();
        registry.registerBeanDefinition("hello", beanDefinition);
    }
}
```

原理：

1. ioc创建对象

2. refresh()-》invokeBeanFactoryPostProcessors(beanFactory);

3. 从容器中获取到所有的BeanDefinitionRegistryPostProcessor组件。
   1. 依次触发所有的postProcessBeanDefinitionRegistry()方法
   2. 再来触发postProcessBeanFactory()方法BeanFactoryPostProcessor；

4. 再来从容器中找到BeanFactoryPostProcessor组件；然后依次触发postProcessBeanFactory()方法

### ApplicationListener

监听容器中发布的事件。事件驱动模型开发；

```java
 public interface ApplicationListener<E extends ApplicationEvent>
      //监听 ApplicationEvent 及其下面的子事件；
```

```java
@Component
public class MyApplicationListener implements ApplicationListener<ApplicationEvent> {
    //当容器中发布此事件以后，方法触发
    @Override
    public void onApplicationEvent(ApplicationEvent event) {
        System.out.println("收到事件："+event);
    }
}
@Test
public void test01(){
    AnnotationConfigApplicationContext applicationContext  =
        new AnnotationConfigApplicationContext(ExtConfig.class);
    //发布事件；
    applicationContext.publishEvent(new ApplicationEvent(new String("我发布的时间")) {
    });
    applicationContext.close();
}
```

####  步骤：

1. 写一个监听器（ApplicationListener实现类）来监听某个事件（ApplicationEvent及其子类）
            @EventListener;
                    原理：使用EventListenerMethodProcessor处理器来解析方法上的@EventListener；

2. 把监听器加入到容器；
3. 只要容器中有相关事件的发布，我们就能监听到这个事件；
           ContextRefreshedEvent：容器刷新完成（所有bean都完全创建）会发布这个事件；
           ContextClosedEvent：关闭容器会发布这个事件；
4.  发布一个事件：
           applicationContext.publishEvent()；

####  原理：

​       ContextRefreshedEvent、IOCTest_Ext$1[source=我发布的时间]、ContextClosedEvent；

1. ContextRefreshedEvent事件：
   1. 容器创建对象：refresh()；
   2. finishRefresh();容器刷新完成会发布ContextRefreshedEvent事件
   3. publishEvent(new ContextRefreshedEvent(this));   -->事件发布流程
2. 自己发布事件；
3. 容器关闭会发布ContextClosedEvent；



【**事件发布流程**】：

1. 获取事件的多播器（派发器）：getApplicationEventMulticaster() -->事件多播器
2. multicastEvent派发事件：
3. 获取到所有的ApplicationListener；
   for (final ApplicationListener<?> listener : getApplicationListeners(event, type)) { -->获取容器中的监听器
   1. 如果有Executor，可以支持使用Executor进行异步派发；
                         Executor executor = getTaskExecutor();
   2. 否则，同步的方式直接执行listener方法；invokeListener(listener, event);
                           拿到listener回调onApplicationEvent方法；

 【**事件多播器（派发器）**】

1. 容器创建对象：refresh();
2. initApplicationEventMulticaster();初始化ApplicationEventMulticaster；
   1. 先去容器中找有没有id=“applicationEventMulticaster”的组件；
   2. 如果没有this.applicationEventMulticaster = new SimpleApplicationEventMulticaster(beanFactory);
       并且加入到容器中，我们就可以在其他组件要派发事件，自动注入这个applicationEventMulticaster；

【**容器中有哪些监听器**】

1. 容器创建对象：refresh();
2. registerListeners();
         从容器中拿到所有的监听器，把他们注册到applicationEventMulticaster中；
          String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
          //将listener注册到ApplicationEventMulticaster中
          getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);

#### @EventListener

```java
@Service
public class UserService {
	@EventListener(classes={ApplicationEvent.class})
	public void listen(ApplicationEvent event){
		System.out.println("UserService。。监听到的事件："+event);
	}
}
```

 原理：使用EventListenerMethodProcessor处理器来解析方法上的@EventListener；

 SmartInitializingSingleton 原理：->afterSingletonsInstantiated();

1. ioc容器创建对象并refresh()；
2. finishBeanFactoryInitialization(beanFactory);初始化剩下的单实例bean；
   1. 先创建所有的单实例bean；getBean();
   2. 获取所有创建好的单实例bean，判断是否是SmartInitializingSingleton类型的；
              如果是就调用afterSingletonsInstantiated();

### Spring容器创建过程

Spring容器的refresh()【创建刷新】

#### 1.prepareRefresh()

刷新前的预处理

1. initPropertySources()初始化一些属性设置;子类自定义个性化的属性设置方法；
2. getEnvironment().validateRequiredProperties();检验属性的合法等
3. earlyApplicationEvents= new LinkedHashSet<ApplicationEvent>();保存容器中的一些早期的事件；

#### 2.obtainFreshBeanFactory()

获取BeanFactory

1. refreshBeanFactory();刷新【创建】BeanFactory；
   			创建了一个this.beanFactory = new DefaultListableBeanFactory();
      			设置id；
2. getBeanFactory();返回刚才GenericApplicationContext创建的BeanFactory对象；
3. 将创建的BeanFactory【DefaultListableBeanFactory】返回；

#### 3.prepareBeanFactory(beanFactory)

BeanFactory的预准备工作（BeanFactory进行一些设置）

1. 设置BeanFactory的类加载器、支持表达式解析器...
2. 添加部分BeanPostProcessor【ApplicationContextAwareProcessor】
3. 设置忽略的自动装配的接口EnvironmentAware、EmbeddedValueResolverAware、xxx；
4. 注册可以解析的自动装配；我们能直接在任何组件中自动注入：
   			BeanFactory、ResourceLoader、ApplicationEventPublisher、ApplicationContext
5. 添加BeanPostProcessor【ApplicationListenerDetector】
6. 添加编译时的AspectJ；
7. 给BeanFactory中注册一些能用的组件；
   		environment【ConfigurableEnvironment】、
      		systemProperties【Map<String, Object>】、
      		systemEnvironment【Map<String, Object>】

#### 4.postProcessBeanFactory(beanFactory)

BeanFactory准备工作完成后进行的后置处理工作

1. 子类通过重写这个方法来在BeanFactory创建并预准备完成以后做进一步的设置

======================以上是BeanFactory的创建及预准备工作=================================================================================================

#### 5.invokeBeanFactoryPostProcessors(beanFactory)

执行BeanFactoryPostProcessor的方法

​	BeanFactoryPostProcessor：BeanFactory的后置处理器。在BeanFactory标准初始化之后执行的；
​	两个接口：BeanFactoryPostProcessor、BeanDefinitionRegistryPostProcessor

1. 执行BeanFactoryPostProcessor的方法；

   先执行BeanDefinitionRegistryPostProcessor

   1. 获取所有的BeanDefinitionRegistryPostProcessor；
   2. 看先执行实现了PriorityOrdered优先级接口的BeanDefinitionRegistryPostProcessor、
      			postProcessor.postProcessBeanDefinitionRegistry(registry)
   3. 在执行实现了Ordered顺序接口的BeanDefinitionRegistryPostProcessor；
      			postProcessor.postProcessBeanDefinitionRegistry(registry)
   4. 最后执行没有实现任何优先级或者是顺序接口的BeanDefinitionRegistryPostProcessors；
      			postProcessor.postProcessBeanDefinitionRegistry(registry)
   
   再执行BeanFactoryPostProcessor的方法
   
   1. 获取所有的BeanFactoryPostProcessor
   2. 看先执行实现了PriorityOrdered优先级接口的BeanFactoryPostProcessor、
            	postProcessor.postProcessBeanFactory()
   3. 在执行实现了Ordered顺序接口的BeanFactoryPostProcessor；
            	postProcessor.postProcessBeanFactory()
   4. 最后执行没有实现任何优先级或者是顺序接口的BeanFactoryPostProcessor；
            	postProcessor.postProcessBeanFactory()

#### 6.registerBeanPostProcessors(beanFactory)

注册BeanPostProcessor（Bean的后置处理器）【 intercept bean creation】

​		不同接口类型的BeanPostProcessor；在Bean创建前后的执行时机是不一样的
​		BeanPostProcessor、
​		DestructionAwareBeanPostProcessor、
​		InstantiationAwareBeanPostProcessor、
​		SmartInstantiationAwareBeanPostProcessor、
​		MergedBeanDefinitionPostProcessor【internalPostProcessors】、
​		

1. 获取所有的 BeanPostProcessor;后置处理器都默认可以通过PriorityOrdered、Ordered接口来执行优先级
2. 先注册PriorityOrdered优先级接口的BeanPostProcessor；
   		把每一个BeanPostProcessor；添加到BeanFactory中
      		beanFactory.addBeanPostProcessor(postProcessor);
3. 再注册Ordered接口的
4. 最后注册没有实现任何优先级接口的
5. 最终注册MergedBeanDefinitionPostProcessor；
6. 注册一个ApplicationListenerDetector；来在Bean创建完成后检查是否是ApplicationListener，如果是
   		applicationContext.addApplicationListener((ApplicationListener<?>) bean);

#### 7.initMessageSource()

初始化MessageSource组件（做国际化功能；消息绑定，消息解析）

1. 获取BeanFactory
2. 看容器中是否有id为messageSource的，类型是MessageSource的组件
   			如果有赋值给messageSource，如果没有自己创建一个DelegatingMessageSource；
      				MessageSource：取出国际化配置文件中的某个key的值；能按照区域信息获取；
3. 把创建好的MessageSource注册在容器中，以后获取国际化配置文件的值的时候，可以自动注入MessageSource；
   			beanFactory.registerSingleton(MESSAGE_SOURCE_BEAN_NAME, this.messageSource);	
      			MessageSource.getMessage(String code, Object[] args, String defaultMessage, Locale locale);

#### 8.initApplicationEventMulticaster()

初始化事件派发器

1. 获取BeanFactory
2. 从BeanFactory中获取applicationEventMulticaster的ApplicationEventMulticaster；
3. 如果上一步没有配置；创建一个SimpleApplicationEventMulticaster
4. 将创建的ApplicationEventMulticaster添加到BeanFactory中，以后其他组件直接自动注入

#### 9.onRefresh()

留给子容器（子类）

1. 子类重写这个方法，在容器刷新的时候可以自定义逻辑；

#### 10.registerListeners()

给容器中将所有项目里面的ApplicationListener注册进来；

1. 从容器中拿到所有的ApplicationListener
2. 将每个监听器添加到事件派发器中；
   			getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
3. 派发之前步骤产生的事件；

#### 11.finishBeanFactoryInitialization(beanFactory)

初始化所有剩下的单实例bean

beanFactory.preInstantiateSingletons();初始化后剩下的单实例bean

1. 获取容器中的所有Bean，依次进行初始化和创建对象

2. 获取Bean的定义信息；RootBeanDefinition

3. Bean不是抽象的，是单实例的，是懒加载；

   1. 判断是否是FactoryBean；是否是实现FactoryBean接口的Bean；

   2. 不是工厂Bean。利用getBean(beanName);创建对象

      0. getBean(beanName)； ioc.getBean();

      1. doGetBean(name, null, null, false);

      2. 先获取缓存中保存的单实例Bean。如果能获取到说明这个Bean之前被创建过（所有创建过的单实例Bean都会被缓存起来）
         从private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);获取的

      3. 缓存中获取不到，开始Bean的创建对象流程；

      4. 标记当前bean已经被创建

      5. 获取Bean的定义信息；

      6. 【获取当前Bean依赖的其他Bean;如果有按照getBean()把依赖的Bean先创建出来；】

      7. 启动单实例Bean的创建流程；

         1. createBean(beanName, mbd, args);

         2. Object bean = resolveBeforeInstantiation(beanName, mbdToUse);

            让BeanPostProcessor先拦截返回代理对象；
            【InstantiationAwareBeanPostProcessor】：提前执行；
            			先触发：postProcessBeforeInstantiation()；
            			如果有返回值：触发postProcessAfterInitialization()；

         3. 如果前面的InstantiationAwareBeanPostProcessor没有返回代理对象；调用4）

         4. Object beanInstance = doCreateBean(beanName, mbdToUse, args);创建Bean

            1. 【创建Bean实例】；createBeanInstance(beanName, mbd, args);
               						 	利用工厂方法或者对象的构造器创建出Bean实例；

            2. applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);

               调用MergedBeanDefinitionPostProcessor的

               postProcessMergedBeanDefinition(mbd, beanType, beanName);

            3. 【Bean属性赋值】populateBean(beanName, mbd, instanceWrapper);
               	赋值之前：

               1. 拿到InstantiationAwareBeanPostProcessor后置处理器；
                  						 		postProcessAfterInstantiation()；

               2. 拿到InstantiationAwareBeanPostProcessor后置处理器；
                  						 		postProcessPropertyValues()；
                  ==赋值之前：==

               3. 应用Bean属性的值；为属性利用setter方法等进行赋值；
                  						 		applyPropertyValues(beanName, mbd, bw, pvs);

               4. 【Bean初始化】initializeBean(beanName, exposedObject, mbd);

                  1. 【执行Aware接口方法】invokeAwareMethods(beanName, bean);

                     执行xxxAware接口的方法
                     BeanNameAware\BeanClassLoaderAware\BeanFactoryAware

                  2. 【执行后置处理器初始化之前】

                     applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
                     BeanPostProcessor.postProcessBeforeInitialization（）;

                  3. 【执行初始化方法】invokeInitMethods(beanName, wrappedBean, mbd);

                     1. 是否是InitializingBean接口的实现；执行接口规定的初始化；
                     2. 是否自定义初始化方法；

                  4. 【执行后置处理器初始化之后】applyBeanPostProcessorsAfterInitialization
                     	BeanPostProcessor.postProcessAfterInitialization()；

                  5. 注册Bean的销毁方法；

               5. 将创建的Bean添加到缓存中singletonObjects；

ioc容器就是这些Map；很多的Map里面保存了单实例Bean，环境信息。。。。；
		    所有Bean都利用getBean创建完成以后；
			检查所有的Bean是否是SmartInitializingSingleton接口的；如果是；就执行afterSingletonsInstantiated()；

#### 12.finishRefresh()

完成BeanFactory的初始化创建工作；IOC容器就创建完成

1. initLifecycleProcessor();初始化和生命周期有关的后置处理器；LifecycleProcessor

   默认从容器中找是否有lifecycleProcessor的组件【LifecycleProcessor】

   如果没有new DefaultLifecycleProcessor();
   			加入到容器；

   写一个LifecycleProcessor的实现类，可以在BeanFactory
   		void onRefresh();
   		void onClose();	

2. getLifecycleProcessor().onRefresh();
   	拿到前面定义的生命周期处理器（BeanFactory）；回调onRefresh()；

3. publishEvent(new ContextRefreshedEvent(this));发布容器刷新完成事件；

4. liveBeansView.registerApplicationContext(this);

#### 总结

1. Spring容器在启动的时候，先会保存所有注册进来的Bean的定义信息；
   1. xml注册bean；<bean>
   2. 注解注册Bean；@Service、@Component、@Bean、xxx
2. Spring容器会合适的时机创建这些Bean
   1. 用到这个bean的时候；利用getBean创建bean；创建好以后保存在容器中；
   2. 统一创建剩下所有的bean的时候；finishBeanFactoryInitialization()；
3. 后置处理器；BeanPostProcessor
   1. 每一个bean创建完成，都会使用各种后置处理器进行处理；来增强bean的功能；
      		AutowiredAnnotationBeanPostProcessor:处理自动注入
         		AnnotationAwareAspectJAutoProxyCreator:来做AOP功能；
         		xxx....
         		增强的功能注解：
         		AsyncAnnotationBeanPostProcessor
         		....
4. 事件驱动模型；
   	ApplicationListener；事件监听；
      	ApplicationEventMulticaster；事件派发：



## web

### Servlet3.0

#### 注解

```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {
	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		//super.doGet(req, resp);
		System.out.println(Thread.currentThread()+" start...");
		try {
			sayHello();
		} catch (Exception e) {
			e.printStackTrace();
		}
		resp.getWriter().write("hello...");
		System.out.println(Thread.currentThread()+" end...");
	}
	public void sayHello() throws Exception{
		System.out.println(Thread.currentThread()+" processing...");
		Thread.sleep(3000);
	}
}
```

#### 运行时插件能力

Shared libraries（共享库） / runtimes pluggability（运行时插件能力）

1. Servlet容器启动会扫描，当前应用里面每一个jar包的
   	ServletContainerInitializer的实现
2. 提供ServletContainerInitializer的实现类；
   	必须绑定在，META-INF/services/javax.servlet.ServletContainerInitializer
      	文件的内容就是ServletContainerInitializer实现类的全类名；

总结：容器在启动应用的时候，会扫描当前应用每一个jar包里面
META-INF/services/javax.servlet.ServletContainerInitializer
指定的实现类，启动并运行这个实现类的方法；传入感兴趣的类型；

ServletContainerInitializer；
@HandlesTypes；		

```java
META-INF/services/javax.servlet.ServletContainerInitializer
com.atguigu.servlet.MyServletContainerInitializer
```

```java
//容器启动的时候会将@HandlesTypes指定的这个类型下面的子类（实现类，子接口等）传递过来；
//传入感兴趣的类型；
@HandlesTypes(value={HelloService.class})
public class MyServletContainerInitializer implements ServletContainerInitializer {
	/**
	 * 应用启动的时候，会运行onStartup方法；
	 * 
	 * Set<Class<?>> arg0：感兴趣的类型的所有子类型；
	 * ServletContext arg1:代表当前Web应用的ServletContext；一个Web应用一个ServletContext；
	 * 
	 * 1）、使用ServletContext注册Web组件（Servlet、Filter、Listener）
	 * 2）、使用编码的方式，在项目启动的时候给ServletContext里面添加组件；
	 * 		必须在项目启动的时候来添加；
	 * 		1）、ServletContainerInitializer得到的ServletContext；
	 * 		2）、ServletContextListener得到的ServletContext；
	 */
	@Override
	public void onStartup(Set<Class<?>> arg0, ServletContext sc) throws ServletException {
		System.out.println("感兴趣的类型：");
		for (Class<?> claz : arg0) {
			System.out.println(claz);
		}
		
		//注册组件  ServletRegistration  
		ServletRegistration.Dynamic servlet = sc.addServlet("userServlet", new UserServlet());
		//配置servlet的映射信息
		servlet.addMapping("/user");
		
		//注册Listener
		sc.addListener(UserListener.class);
		
		//注册Filter  FilterRegistration
		FilterRegistration.Dynamic filter = sc.addFilter("userFilter", UserFilter.class);
		//配置Filter的映射信息
		filter.addMappingForUrlPatterns(EnumSet.of(DispatcherType.REQUEST), true, "/*");
	}
}
```

```java
public class UserServlet extends HttpServlet {
	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		resp.getWriter().write("tomcat...");
	}
}
/**
 * 监听项目的启动和停止
 */
public class UserListener implements ServletContextListener {
	//监听ServletContext销毁
	@Override
	public void contextDestroyed(ServletContextEvent arg0) {
		System.out.println("UserListener...contextDestroyed...");
	}
	//监听ServletContext启动初始化
	@Override
	public void contextInitialized(ServletContextEvent arg0) {
		ServletContext servletContext = arg0.getServletContext();
		System.out.println("UserListener...contextInitialized...");
	}
}
public class UserFilter implements Filter {
	@Override
	public void destroy() {	
	}
	@Override
	public void doFilter(ServletRequest arg0, ServletResponse arg1, FilterChain arg2)
			throws IOException, ServletException {
		// 过滤请求
		System.out.println("UserFilter...doFilter...");
		//放行
		arg2.doFilter(arg0, arg1);
	}
	@Override
	public void init(FilterConfig arg0) throws ServletException {
	}
}
```

​			 	

### Spring MVC	

1、web容器在启动的时候，会扫描每个jar包下的META-INF/services/javax.servlet.ServletContainerInitializer
2、加载这个文件指定的类SpringServletContainerInitializer
3、spring的应用一启动会加载感兴趣的WebApplicationInitializer接口的下的所有组件；
4、并且为WebApplicationInitializer组件创建对象（组件不是接口，不是抽象类）
	1）、AbstractContextLoaderInitializer：创建根容器；createRootApplicationContext()；
	2）、AbstractDispatcherServletInitializer：
			创建一个web的ioc容器；createServletApplicationContext();
			创建了DispatcherServlet；createDispatcherServlet()；
			将创建的DispatcherServlet添加到ServletContext中；
				getServletMappings();
	3）、AbstractAnnotationConfigDispatcherServletInitializer：注解方式配置的DispatcherServlet初始化器
			创建根容器：createRootApplicationContext()
					getRootConfigClasses();传入一个配置类
			创建web的ioc容器： createServletApplicationContext();
					获取配置类；getServletConfigClasses();
	

#### 总结

​	以注解方式来启动SpringMVC；继承AbstractAnnotationConfigDispatcherServletInitializer；
实现抽象方法指定DispatcherServlet的配置信息；



#### 定制SpringMVC

1）、@EnableWebMvc:开启SpringMVC定制配置功能；

```xml
<mvc:annotation-driven/>
```

2）、配置组件（视图解析器、视图映射、静态资源映射、拦截器。。。）
	extends WebMvcConfigurerAdapter

#### 整合

```java
//web容器启动的时候创建对象；调用方法来初始化容器以前前端控制器
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

	//获取根容器的配置类；（Spring的配置文件）   父容器；
	@Override
	protected Class<?>[] getRootConfigClasses() {
		// TODO Auto-generated method stub
		return new Class<?>[]{RootConfig.class};
	}

	//获取web容器的配置类（SpringMVC配置文件）  子容器；
	@Override
	protected Class<?>[] getServletConfigClasses() {
		return new Class<?>[]{AppConfig.class};
	}

	//获取DispatcherServlet的映射信息
	//  /：拦截所有请求（包括静态资源（xx.js,xx.png）），但是不包括*.jsp；
	//  /*：拦截所有请求；连*.jsp页面都拦截；jsp页面是tomcat的jsp引擎解析的；
	@Override
	protected String[] getServletMappings() {
		return new String[]{"/"};
	}

}
```

```java
//Spring的容器不扫描controller;父容器
@ComponentScan(value="com.atguigu",excludeFilters={
    @Filter(type=FilterType.ANNOTATION,classes={Controller.class})
})
public class RootConfig {
}
//SpringMVC只扫描Controller；子容器
//useDefaultFilters=false 禁用默认的过滤规则；
@ComponentScan(value="com.atguigu",includeFilters={
    @Filter(type=FilterType.ANNOTATION,classes={Controller.class})
},useDefaultFilters=false)
@EnableWebMvc
public class AppConfig  extends WebMvcConfigurerAdapter  {

    //定制
    //视图解析器
    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        //默认所有的页面都从 /WEB-INF/ xxx .jsp
        //registry.jsp();
        registry.jsp("/WEB-INF/views/", ".jsp");
    }

    //静态资源访问
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    //拦截器
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //super.addInterceptors(registry);
        registry.addInterceptor(new MyFirstInterceptor()).addPathPatterns("/**");
    }
}
```

```java
public class MyFirstInterceptor implements HandlerInterceptor {//拦截器

	//目标方法运行之前执行
	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response
                             , Object handler) throws Exception {
		System.out.println("preHandle..."+request.getRequestURI());
		return true;
	}
	//目标方法执行正确以后执行
	@Override
	public void postHandle(HttpServletRequest request, HttpServletResponse response
                           , Object handler, ModelAndView modelAndView) throws Exception {
		System.out.println("postHandle...");
	}
	//页面响应以后执行
	@Override
	public void afterCompletion(HttpServletRequest request, HttpServletResponse response
                                , Object handler, Exception ex)throws Exception {
		System.out.println("afterCompletion...");
	}

}
```

### 异步处理

#### Servlet3.0

```java
@WebServlet(value="/async",asyncSupported=true)
public class HelloAsyncServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) 
        throws ServletException, IOException {
        //1、支持异步处理asyncSupported=true
        //2、开启异步模式
        System.out.println("主线程开始。。。"+
                           Thread.currentThread()+"==>"+System.currentTimeMillis());
        AsyncContext startAsync = req.startAsync();

        //3、业务逻辑进行异步处理;开始异步处理
        startAsync.start(new Runnable() {
            @Override
            public void run() {
                try {
                    System.out.println("副线程开始。。。"+
                                       Thread.currentThread()+"==>"+System.currentTimeMillis());
                    sayHello();
                    startAsync.complete();
                    //获取到异步上下文
                    AsyncContext asyncContext = req.getAsyncContext();
                    //4、获取响应
                    ServletResponse response = asyncContext.getResponse();
                    response.getWriter().write("hello async...");
                    System.out.println("副线程结束。。。"+
                                       Thread.currentThread()+"==>"+System.currentTimeMillis());
                } catch (Exception e) {
                }
            }
        });		
        System.out.println("主线程结束。。。"+
                           Thread.currentThread()+"==>"+System.currentTimeMillis());
    }

    public void sayHello() throws Exception{
        System.out.println(Thread.currentThread()+" processing...");
        Thread.sleep(3000);
    }
}
```

#### spring mvc

1. 控制器返回Callable
2. Spring异步处理，将Callable 提交到 TaskExecutor 使用一个隔离的线程进行执行
3. DispatcherServlet和所有的Filter退出web容器的线程，但是response 保持打开状态；
4. Callable返回结果，SpringMVC将请求重新派发给容器，恢复之前的处理；
5. 根据Callable返回的结果。SpringMVC继续进行视图渲染流程等（从收请求-视图渲染）。

异步的拦截器:

1. 原生API的AsyncListener
2. SpringMVC：实现AsyncHandlerInterceptor；

```java
@Controller
public class AsyncController {
    @ResponseBody//模拟消息中间件异步调用
    @RequestMapping("/createOrder")
    public DeferredResult<Object> createOrder(){
        DeferredResult<Object> deferredResult = new DeferredResult<>((long)3000
                                                                     , "create fail...");
        DeferredResultQueue.save(deferredResult);
        return deferredResult;
    }
    @ResponseBody
    @RequestMapping("/create")
    public String create(){
        //创建订单
        String order = UUID.randomUUID().toString();
        DeferredResult<Object> deferredResult = DeferredResultQueue.get();
        deferredResult.setResult(order);//设置返回
        return "success===>"+order;
    }
    /**
	 * preHandle.../springmvc-annotation/async01
		主线程开始...Thread[http-bio-8081-exec-3,5,main]==>1513932494700
		主线程结束...Thread[http-bio-8081-exec-3,5,main]==>1513932494700
		=========DispatcherServlet及所有的Filter退出线程============================

		================等待Callable执行==========
		副线程开始...Thread[MvcAsync1,5,main]==>1513932494707
		副线程开始...Thread[MvcAsync1,5,main]==>1513932496708
		================Callable执行完成==========

		================再次收到之前重发过来的请求========
		preHandle.../springmvc-annotation/async01
		postHandle...（Callable的之前的返回值就是目标方法的返回值）
		afterCompletion...
	 */
    @ResponseBody
    @RequestMapping("/async01")
    public Callable<String> async01(){
        System.out.println("主线程开始..."+
                           Thread.currentThread()+"==>"+System.currentTimeMillis());

        Callable<String> callable = new Callable<String>() {
            @Override
            public String call() throws Exception {
                System.out.println("副线程开始..."+Thread.currentThread()
                                   +"==>"+System.currentTimeMillis());
                Thread.sleep(2000);
                System.out.println("副线程开始..."+Thread.currentThread()+"==>"
                                   +System.currentTimeMillis());
                return "Callable<String> async01()";
            }
        };

        System.out.println("主线程结束..."+Thread.currentThread()+
                           "==>"+System.currentTimeMillis());
        return callable;
    }
}
```

# Spring5

轻量级的开源的JavaEE框架

1. IOC：控制反转，把创建对象过程交给Spring进行管理
2. Aop：面向切面，不修改源代码进行功能增强

Spring特点

1. 方便解耦，简化开发
2. Aop编程支持
3. 方便程序测试
4. 方便和其他框架进行整合
5. 方便进行事务操作
6. 降低API开发难度

下载地址https://repo.spring.io/release/org/springframework/spring/

```java
//1 加载spring配置文件 
ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml"); 
//2 获取配置创建的对象 
User user = context.getBean("user", User.class); 
System.out.println(user); 
```

## IOC容器

控制反转，把对象创建和对象之间的调用过程，交给Spring进行管理
降低耦合度



### **IOC底层原理**

xml解析、工厂模式、反射

1. 读取xml配置
2. 创建工厂类通过反射创建对象

#### IOC(BeanFactory 接口)

1、IOC思想基于IOC容器完成，IOC容器底层就是对象工厂

2、Spring提供IOC容器实现两种方式：（两个接口）

1. BeanFactory：IOC容器基本实现，是Spring内部的使用接口，不提供开发人员进行使用.

    加载配置文件时候不会创建对象，在获取对象（使用）才去创建对象

2. ApplicationContext：BeanFactory接口的子接口，提供更多更强大的功能，一般由开发人员进行使用

   加载配置文件时候就会把在配置文件对象进行创建
   

3、ApplicationContext接口有实现类

![image-20210819133237105](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210819133237105.png)

```java
AnnotationConfigApplicationContext
GroovyWebApplicationContext
XmlWebApplicationContext
ClassPathXmlApplicationContext
FileSystemXmlApplicationContext
```

### Bean管理

创建对象,注入属性

#### 1.xml

##### 1.基于xml方式创建对象

```xml
<bean id="唯一标识" class="类全路径" name="类似id"></bean>
```

创建对象时候，默认也是执行无参数构造方法完成对象创建



##### 2.基于xml方式注入属性

​	DI：依赖注入，就是注入属性



###### 1.第一种注入方式：set方法注入

1. 创建类，定义属性和对应的set方法

2. 在spring配置文件配置对象创建，配置属性注入

   ```xml
   <bean id="" class="">
       <property name="属性名" value="值"></property>
   </bean>
   ```

###### 2.第二种注入方式：有参数构造注入

1. 创建类，定义属性，创建属性对应有参数构造方法

2. 在spring配置文件中进行配置

   ```xml
   <bean id="" class="">
       <constructor-arg name="属性" value="值"></constructor-arg>
       <constructor-arg index="第几个参数 0开始" value="值"></constructor-arg>
   </bean>
   ```

###### 3.p名称空间注入（了解）

（1）使用p名称空间注入，可以简化基于xml配置方式
第一步 添加p名称空间在配置文件中

```xml 
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"     //添加
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
```

第二步 进行属性注入，在bean标签里面进行操作

```xml
<bean id="" class="" p:bname="值" p:bauthor="值"></bean>
```

###### 4.注入其他类型属性

1. null值

   ```xml
   <property name="address">
       <null/>
   </property>
   ```

   特殊字符

   ```xml
   //把<>进行转义 &lt; &gt;
   //把带特殊符号内容写到CDATA
   <property name="address">
       <value><![CDATA[特殊符号]]></value>
   </property>
   ```

2. 外部bean

   service调dao

   ```java
   public class UserService {
       //创建UserDao类型属性，生成set方法
       private UserDao userDao;
       public void setUserDao(UserDao userDao) {
           this.userDao = userDao;
       }
       public void add() {
           System.out.println("service add...............");
           userDao.update();
       }
   }
   ```

   ```xml
   <!--1 service和dao对象创建-->
   <bean id="userService" class="com.atguigu.spring5.service.UserService">
       <!--注入userDao对象
               name属性：类里面属性名称
               ref属性：创建userDao对象bean标签id值-->
       <property name="userDao" ref="userDaoImpl"></property>
   </bean>
   <bean id="userDaoImpl" class="com.atguigu.spring5.dao.UserDaoImpl"></bean>
   ```

3. 内部bean

   ```java
   //一对多
   //员工类
   public class Emp {
       private String ename;
       private String gender;
       //员工属于某一个部门，使用对象形式表示
       private Dept dept;
   }
   ```

   ```xml
   <!--内部bean-->
   <bean id="emp" class="com.atguigu.spring5.bean.Emp">
       <!--设置两个普通属性-->
       <property name="ename" value="lucy"></property>
       <property name="gender" value="女"></property>
       <!--设置对象类型属性-->
       <property name="dept">
           <bean id="dept" class="com.atguigu.spring5.bean.Dept">
               <property name="dname" value="安保部"></property>
           </bean>
       </property>
   </bean>
   ```

4. 级联赋值

   ```xml
   <!--级联赋值-->
   <bean id="emp" class="com.atguigu.spring5.bean.Emp">
       <!--设置两个普通属性-->
       <property name="ename" value="lucy"></property>
       <property name="gender" value="女"></property>
       <!--级联赋值-->
       <property name="dept" ref="dept"></property><!--写法一-->
       <property name="dept.dname" value="技术部"></property><!--写法二-->
   </bean>
   <bean id="dept" class="com.atguigu.spring5.bean.Dept">
       <property name="dname" value="财务部"></property>
   </bean>
   ```

###### 5.注入集合

数组 list map set

```xml
<!--1 集合类型属性注入-->
<bean id="stu" class="com.atguigu.spring5.collectiontype.Stu">
    <!--数组类型属性注入-->
    <property name="courses">
        <array>
            <value>java课程</value>
            <value>数据库课程</value>
        </array>
    </property>
    <!--list类型属性注入-->
    <property name="list">
        <list>
            <value>张三</value>
            <value>小三</value>
        </list>
    </property>
    <!--map类型属性注入-->
    <property name="maps">
        <map>
            <entry key="JAVA" value="java"></entry>
            <entry key="PHP" value="php"></entry>
        </map>
    </property>
    <!--set类型属性注入-->
    <property name="sets">
        <set>
            <value>MySQL</value>
            <value>Redis</value>
        </set>
    </property>
    <!--注入list集合类型，值是对象-->
    <property name="courseList">
        <list>
            <ref bean="course1"></ref>
            <ref bean="course2"></ref>
        </list>
    </property>
</bean>

<!--创建多个course对象-->
<bean id="course1" class="com.atguigu.spring5.collectiontype.Course">
    <property name="cname" value="Spring5框架"></property>
</bean>
<bean id="course2" class="com.atguigu.spring5.collectiontype.Course">
    <property name="cname" value="MyBatis框架"></property>
</bean>
```

把集合注入部分提取出来

1. 在spring配置文件中引入名称空间 util

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:p="http://www.springframework.org/schema/p"
          xmlns:util="http://www.springframework.org/schema/util"//改
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                              http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">//改
   ```

2. 使用util标签完成list集合注入提取

   ```xml
   <!--1 提取list集合类型属性注入-->
   <util:list id="bookList">
       <value>易筋经</value>
       <value>九阴真经</value>
       <value>九阳神功</value>
   </util:list>
   <!--2 提取list集合类型属性注入使用-->
   <bean id="book" class="com.atguigu.spring5.collectiontype.Book" scope="prototype">
       <property name="list" ref="bookList"></property>
   </bean>
   ```

##### 3.FactoryBean

Spring有两种类型bean，一种普通bean，另外一种工厂bean（FactoryBean）

* 普通bean：在配置文件中定义bean类型就是返回类型
* 工厂bean：在配置文件定义bean类型可以和返回类型不一样

第一步 创建类，让这个类作为工厂bean，实现接口 FactoryBean
第二步 实现接口里面的方法，在实现的方法中定义返回的bean类型

```java
public class MyBean implements FactoryBean<Course> {
    //定义返回bean
    @Override
    public Course getObject() throws Exception {
        Course course = new Course();
        course.setCname("abc");
        return course;
    }
    @Override
    public Class<?> getObjectType() {
        return null;
    }
    @Override
    public boolean isSingleton() {
        return false;
    }
}
```

##### 4.Bean的作用域

默认情况下，bean是单实例对象

```xml
<bean id="" class="" scope="prototype多实例">
    <property name="" ref=""></property>
</bean>
```

singleton，加载spring配置文件时候就会创建单实例对象
prototype，调用getBean方法时候创建多实例对象

request 放入request

session  放入session

##### 5.Bean生命周期

（1）通过构造器创建bean实例（无参数构造）
（2）为bean的属性设置值和对其他bean引用（调用set方法）
（3）把bean实例传递bean后置处理器的方法postProcessBeforeInitialization
（4）调用bean的初始化的方法（需要进行配置初始化的方法）
（5）把bean实例传递bean后置处理器的方法 postProcessAfterInitialization
（6）bean可以使用了（对象获取到了）
（7）当容器关闭时候，调用bean的销毁的方法（需要进行配置销毁的方法）

```xml
<bean id="" class="" init-method="initMethod" destroy-method="destroyMethod">
    <property name="oname" value="手机"></property>
</bean>
<!--配置后置处理器-->
<bean id="myBeanPost" class="com.atguigu.spring5.bean.MyBeanPost"></bean>
```

```java
public class Orders {
    //无参数构造
    public Orders() {
        System.out.println("第一步 执行无参数构造创建bean实例");
    }
    private String oname;
    public void setOname(String oname) {
        this.oname = oname;
        System.out.println("第二步 调用set方法设置属性值");
    }
    //创建执行的初始化的方法
    public void initMethod() {
        System.out.println("第三步 执行初始化的方法");
    }
    //创建执行的销毁的方法
    public void destroyMethod() {
        System.out.println("第五步 执行销毁的方法");
    }
}

public class MyBeanPost implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) 
        throws BeansException {
        System.out.println("在初始化之前执行的方法");
        return bean;
    }
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) 
        throws BeansException {
        System.out.println("在初始化之后执行的方法");
        return bean;
    }
}

@Test
public void testBean3() {
    //        ApplicationContext context =
    //                new ClassPathXmlApplicationContext("bean4.xml");
    ClassPathXmlApplicationContext context =
        new ClassPathXmlApplicationContext("bean4.xml");
    Orders orders = context.getBean("orders", Orders.class);
    System.out.println("第四步 获取创建bean实例对象");
    System.out.println(orders);

    //手动让bean实例销毁
    context.close();
}
```

##### 6.自动装配

根据指定装配规则（属性名称或者属性类型），Spring自动将匹配的属性值进行注入

```xml
 <!--实现自动装配
        bean标签属性autowire，配置自动装配
        autowire属性常用两个值：
            byName根据属性名称注入 ，注入值bean的id值和类属性名称一样
            byType根据属性类型注入
    -->
    <bean id="emp" class="com.atguigu.spring5.autowire.Emp" autowire="byType">
        <!--<property name="dept" ref="dept"></property>-->
    </bean>
    <bean id="dept" class="com.atguigu.spring5.autowire.Dept"></bean>
```

##### 7.外部属性文件

配置数据库信息

```xml
<!--引入context名称空间-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:util="http://www.springframework.org/schema/util"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

<!--引入外部属性文件-->
<context:property-placeholder location="classpath:jdbc.properties"/>

<!--配置连接池-->
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value="${prop.driverClass}"></property>
    <property name="url" value="${prop.url}"></property>
    <property name="username" value="${prop.userName}"></property>
    <property name="password" value="${prop.password}"></property>
</bean>
```

```properties
prop.driverClass=com.mysql.jdbc.Driver
prop.url=jdbc:mysql://localhost:3306/userDb
prop.userName=root
prop.password=root
```

#### 2.Annotation

创建对象

```java
@Component
@Service
@Controller
@Repository
```

这四个注解功能一样



##### 1.基于Annotation方式创建对象

开启组件扫描

```xml
<context:component-scan base-package="com.atguigu"></context:component-scan> 
```

添加注解

```java
@Component(value = "abc")//不写默认类名首字母小写
```

##### 2.扫包

```xml
<!--示例1 use-default-filters="false" 表示现在不使用默认filter，自己配置filter context:include-filter ，设置扫描哪些内容 -->
<context:component-scan base-package="com.atguigu" use-default-filters="false"> 
<context:include-filter type="annotation"expression="org.springframework.stereotype.Controller"/> 
</context:component-scan> 
<!--示例2 下面配置扫描包所有内容 context:exclude-filter： 设置哪些内容不进行扫描 --> 
<context:component-scan base-package="com.atguigu"> 
<context:exclude-filter type="annotation"expression="org.springframework.stereotype.Controller"/> 
</context:component-scan>
```

##### 3.基于注解方式实现属性注入

1. @Autowired：根据属性类型进行自动装配

2. @Qualifier：根据名称进行注入
   这个@Qualifier注解的使用，和上面@Autowired一起使用 

   ```java
   @Autowired  //根据类型进行注入
   @Qualifier(value = "userDaoImpl1") //根据名称进行注入
   private UserDao userDao;
   ```

3. @Resource

   ```java
   //@Resource  //根据类型进行注入
   @Resource(name = "userDaoImpl1")  //根据名称进行注入
   private UserDao userDao;
   ```

4. @Value

##### 4.全注解开发

```java
@Configuration  //作为配置类，替代xml配置文件
@ComponentScan(basePackages = {"com.atguigu"})
public class SpringConfig {
}
```

## AOP

面向切面编程

利用 AOP 可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率



### 底层原理

第一种 有接口情况，使用JDK动态代理

创建接口实现类代理对象，增强类的方法



第二种 没有接口情况，使用CGLIB动态代理

创建子类的代理对象，增强类的方法

#### JDK 动态代理

1、使用JDK动态代理，使用Proxy类里面的方法创建代理对象

（1）调用newProxyInstance方法

```java
@CallerSensitive
public static Object newProxyInstance(ClassLoader loader,//类加载器
                                      Class<?>[] interfaces,
                                      //增强方法所在的类，这个类实现的接口，支持多个接口
                                      InvocationHandler h)
    								  //实现这个接口InvocationHandler，创建代理对象，写增强的部分
    throws IllegalArgumentException
```

2、编写JDK动态代理代

接口+实现类

```java
public interface UserDao {
    public int add(int a,int b);
    public String update(String id);
}
public class UserDaoImpl implements UserDao {
    @Override
    public int add(int a, int b) {
        System.out.println("add方法执行了.....");
        return a+b;
    }

    @Override
    public String update(String id) {
        System.out.println("update方法执行了.....");
        return id;
    }
}
```

使用 Proxy 类创建接口代理对象

```java
public class JDKProxy {
    public static void main(String[] args) {
        //创建接口实现类代理对象
        Class[] interfaces = {UserDao.class};
        //        Proxy.newProxyInstance(JDKProxy.class.getClassLoader(), interfaces, new InvocationHandler() {
        //            @Override
        //            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //                return null;
        //            }
        //        });
        UserDaoImpl userDao = new UserDaoImpl();
        UserDao dao = 
            (UserDao)Proxy.newProxyInstance(JDKProxy.class.getClassLoader()
                                            , interfaces, new UserDaoProxy(userDao));
        int result = dao.add(1, 2);
        System.out.println("result:"+result);
    }
}
//创建代理对象代码
class UserDaoProxy implements InvocationHandler {
    //1 把创建的是谁的代理对象，把谁传递过来
    //有参数构造传递
    private Object obj;
    public UserDaoProxy(Object obj) {
        this.obj = obj;
    }

    //增强的逻辑
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //方法之前
        System.out.println("方法之前执行...."+method.getName()
                           +" :传递的参数..."+ Arrays.toString(args));
        //被增强的方法执行
        Object res = method.invoke(obj, args);
        //方法之后
        System.out.println("方法之后执行...."+obj);
        return res;
    }
}
```

### aop术语

1. 连接点:  被增强的方法

2. 切入点:  实际被增强的方法

3. 通知/增强: 实际被增强的部分

   前置

   后置

   环绕

   异常

   最终

4. 切面: 通知应用到切入点的过程

### aop操作

Spring框架一般都是基于AspectJ实现AOP操作

AspectJ不是Spring组成部分，是独立AOP框架，一般把AspectJ和Spirng框架一起使用，进行AOP操作

基于AspectJ实现AOP操作

#### 切入点表达式

作用：知道对哪个类里面的哪个方法进行增强

语法结构： 

```shell
execution([权限修饰符] [返回类型] [类全路径] [方法名称]([参数列表]) )
```

```xml
举例1：对com.atguigu.dao.BookDao类里面的add进行增强
execution(* com.atguigu.dao.BookDao.add(..))

举例2：对com.atguigu.dao.BookDao类里面的所有的方法进行增强
execution(* com.atguigu.dao.BookDao.* (..))

举例3：对com.atguigu.dao包里面所有类，类里面所有方法进行增强
execution(* com.atguigu.dao.*.* (..))
```

#### 基于xml配置文件实现

创建两个类，增强类和被增强类，创建方法
在spring配置文件中创建两个对象,配置切入点

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                        http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
    
<!--创建对象-->
<bean id="book" class="com.atguigu.spring5.aopxml.Book"></bean>
<bean id="bookProxy" class="com.atguigu.spring5.aopxml.BookProxy"></bean>

<!--配置aop增强-->
<aop:config>
    <!--切入点-->
    <aop:pointcut id="p" expression="execution(* com.atguigu.spring5.aopxml.Book.buy(..))"/>
    <!--配置切面-->
    <aop:aspect ref="bookProxy">
        <!--增强作用在具体的方法上-->
        <aop:before method="before" pointcut-ref="p"/>
    </aop:aspect>
</aop:config>
```

#### 基于注解方式实现（使用）

```java
//被增强的类
@Component
public class User {
    public void add() {
        int i = 10/0;
        System.out.println("add.......");
    }
}
```

```java
//增强的类
@Component
@Aspect  //生成代理对象
@Order(3)//数字越小,优先级越高
public class UserProxy {

    //相同切入点抽取
    @Pointcut(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void pointdemo() {

    }

    //前置通知
    //@Before注解表示作为前置通知
    @Before(value = "pointdemo()")
    public void before() {
        System.out.println("before.........");
    }

    //后置通知（返回通知）
    @AfterReturning(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void afterReturning() {
        System.out.println("afterReturning.........");
    }

    //最终通知
    @After(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void after() {
        System.out.println("after.........");
    }

    //异常通知
    @AfterThrowing(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void afterThrowing() {
        System.out.println("afterThrowing.........");
    }

    //环绕通知
    @Around(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("环绕之前.........");
        //被增强的方法执行
        proceedingJoinPoint.proceed();
        System.out.println("环绕之后.........");
    }
}

```

开启注解扫描

```java
@Configuration
@ComponentScan(basePackages = {"com.atguigu"})
@EnableAspectJAutoProxy(proxyTargetClass = true)
public class ConfigAop {
}
```

或

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                        http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

<!-- 开启注解扫描 -->
<context:component-scan base-package="com.atguigu.spring5.aopanno"></context:component-scan>

<!-- 开启Aspect生成代理对象-->
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```

## JdbcTemplate

对jdbc封装



配置连接池, 配置JdbcTemplate对象，注入DataSource

```xml
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close"> <property name="url" value="jdbc:mysql:///user_db" /> 
    <property name="username" value="root" /> 
    <property name="password" value="root" /> 
    <property name="driverClassName" value="com.mysql.jdbc.Driver" /> 
</bean>
<!-- JdbcTemplate对象 --> 
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate"> 
    <!--注入dataSource--> 
    <property name="dataSource" ref="dataSource"></property> 
</bean>
<!-- 组件扫描 -->
<context:component-scan base-package="com.atguigu"></context:component-scan>
```

```java
@Autowired     
private JdbcTemplate jdbcTemplate; 
```

```java
jdbcTemplate.update(sql,param1,param2);
jdbcTemplate.qureyForObject(sql,result.class)
```

```java
Integer count = jdbcTemplate.queryForObject(sql, Integer.class); 
Book book = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<Book>(Book.class), id); 
List<Book> bookList = jdbcTemplate.query(sql, new BeanPropertyRowMapper<Book>(Book.class)); 
//批量操作
jdbcTemplate.batchUpdate(sql, list);//批量增删改

```

## 事务

原子一直持久隔离(acid)



事务添加到 JavaEE 三层结构里面 Service 层（业务逻辑层） 
在 Spring 进行事务管理操作 有两种方式：

1. 编程式事务管理

2. 声明式事务管理(底层使用 AOP 原理 )



Spring 事务管理 API 

提供一个接口，代表事务管理器，这个接口针对不同的框架提供不同的实现类

![image-20210819171217476](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210819171217476.png)



### XML声明式事务管理

在spring配置文件中进行配置
第一步 配置事务管理器

第二步 配置通知

第三步 配置切入点和切面

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
                           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!-- 组件扫描 -->
    <context:component-scan base-package="com.atguigu"></context:component-scan>

    <!-- 数据库连接池 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
          destroy-method="close">
        <property name="url" value="jdbc:mysql:///user_db" />
        <property name="username" value="root" />
        <property name="password" value="root" />
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
    </bean>

    <!-- JdbcTemplate对象 -->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <!--注入dataSource-->
        <property name="dataSource" ref="dataSource"></property>
    </bean>

    <!--1 创建事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--注入数据源-->
        <property name="dataSource" ref="dataSource"></property>
    </bean>

    <!--2 配置通知-->
    <tx:advice id="txadvice">
        <!--配置事务参数-->
        <tx:attributes>
            <!--指定哪种规则的方法上面添加事务-->
            <tx:method name="accountMoney" propagation="REQUIRED"/>
            <!--<tx:method name="account*"/>-->
        </tx:attributes>
    </tx:advice>

    <!--3 配置切入点和切面-->
    <aop:config>
        <!--配置切入点-->
        <aop:pointcut id="pt" expression="execution(* com.atguigu.spring5.service.UserService.*(..))"/>
        <!--配置切面-->
        <aop:advisor advice-ref="txadvice" pointcut-ref="pt"/>
    </aop:config>
</beans>
```

### 注解声明式事务管理

1、在 spring 配置文件配置事务管理器 

```xml
<bean id="transactionManager" 
      class="org.springframework.jdbc.datasource.DataSourceTransactionManager"> 
    <property name="dataSource" ref="dataSource"></property> 
</bean> 
```

开启事务注解 

```xml
<tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven> 
```

或

```java
@Configuration //配置类
@ComponentScan(basePackages = "com.atguigu") //组件扫描
@EnableTransactionManagement //开启事务
public class TxConfig {

    //创建数据库连接池
    @Bean
    public DruidDataSource getDruidDataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql:///user_db");
        dataSource.setUsername("root");
        dataSource.setPassword("root");
        return dataSource;
    }

    //创建JdbcTemplate对象
    @Bean
    public JdbcTemplate getJdbcTemplate(DataSource dataSource) {
        //到ioc容器中根据类型找到dataSource
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        //注入dataSource
        jdbcTemplate.setDataSource(dataSource);
        return jdbcTemplate;
    }

    //创建事务管理器
    @Bean
    public DataSourceTransactionManager getDataSourceTransactionManager(DataSource dataSource) {
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
        transactionManager.setDataSource(dataSource);
        return transactionManager;
    }
}

```

### @Transactional

这个注解添加到类上面，也可以添加方法上面

如果把这个注解添加类上面，这个类里面所有的方法都添加事务
如果把这个注解添加方法上面，为这个方法添加事务



#### 参数

##### propagation：事务传播行为 

多事务方法直接进行调用，这个过程中事务 是如何进行管理的

Spring事务七种传播行为

```java
REQUIRED(0), a调b 有事务就用同一个 没有就新建一个事务
SUPPORTS(1),  有事务就用 没事务就不用
MANDATORY(2),  必须用事务,没有就抛异常
REQUIRES_NEW(3), a调b b新建事务
NOT_SUPPORTED(4), 不用事务,有事务,挂起
NEVER(5),  不用事务,有就抛异常
NESTED(6);有事务就嵌套,否则起新事务
```

##### ioslation：事务隔离级别 

事务有特性成为隔离性，多事务操作之间不会产生影响。不考虑隔离性产生很多问题
有三个读问题：脏读、不可重复读、虚（幻）读

* 脏读：一个未提交事务读取到另一个未提交事务的数据

* 不可重复读：一个未提交事务读取到另一事务修改数据 
* 虚/幻读：一个未提交事务读取到另一提交事务添加数据

通过设置事务隔离级别，解决读问题

read uncommitted 读未提交     脏读、不可重复读、虚（幻）读

read committed 读未提交         不可重复读、虚（幻）读

repeatable read 可重复读         虚（幻）读

serializable 串行化

##### timeout：超时时间

事务需要在一定时间内进行提交，如果不提交进行回滚
默认值是 -1 ，设置时间以秒单位进行计算

##### readOnly：是否只读

读：查询操作，写：添加修改删除操作
readOnly默认值false，表示可以查询，可以添加修改删除操作
设置readOnly值是true，设置成true之后，只能查询

##### rollbackFor：回滚

设置出现哪些异常进行事务回滚

##### noRollbackFor：不回滚

设置出现哪些异常不进行事务回滚

## 事务失效

https://www.cnblogs.com/linyb-geek/p/15866577.html

1. ##### service没有托管给spring

   spring事务生效的前提是，service必须是一个bean对象

2. ##### 抛出受检异常 

   spring默认只会回滚非检查异常和error异常  配置rollbackFor

3. ##### 业务自己捕获了异常

   spring事务只有捕捉到了业务抛出去的异常

   将异常原样抛出；
   设置TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();

4. ##### 切面顺序导致

   spring事务切面的优先级顺序最低，但如果自定义的切面优先级和他一样，且自定义的切面没有正确处理异常，则会同业务自己捕获异常的那种场景一样

   在切面中将异常原样抛出；
   在切面中设置TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();

5. ##### 非public方法

   1、将方法改为public；
   2、修改TansactionAttributeSource,将publicMethodsOnly改为false【这个从源码跟踪得出结论】
   3、开启 AspectJ 代理模式【从spring文档得出结论】

6. ##### 父子容器

   子容器扫描范围过大，将未加事务配置的serivce扫描进来
   1、父子容器个扫个的范围；
   2、不用父子容器，所有bean都交给同一容器管理

7. ##### 方法用final或static修饰

   因为spring事务是用动态代理实现，因此如果方法使用了final修饰，则代理类无法对目标方法进行重写，植入事务功能

   方法不要用final修饰

8. ##### 调用本类方法

   本类方法不经过代理，无法进行增强
   1、注入自己来调用；
   2、使用@EnableAspectJAutoProxy(exposeProxy = true) + AopContext.currentProxy()

9. ##### 多线程调用

   因为spring的事务是通过数据库连接来实现，而数据库连接spring是放在threadLocal里面。同一个事务，只能用同一个数据库连接。而多线程场景下，拿到的数据库连接是不一样的，即是属于不同事务

10. ##### 错误的传播行为

    用的传播特性不支持事务

11. ##### 使用了不支持事务的存储引擎

     使用了不支持事务的存储引擎。比如mysql中的MyISAM

12. ##### 数据源没有配置事务管理器

    因为springboot，他默认已经开启事务管理器。

    org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration

13. ##### 被代理的类过早实例化

    当代理类的实例化早于AbstractAutoProxyCreator后置处理器，就无法被AbstractAutoProxyCreator后置处理器增强

    









## 新功能

整个Spring5框架的代码基于Java8，运行时兼容JDK9，许多不建议使用的类和方法在代码库中删除

### 自带通用的日志封装

1. Spring5已经移除Log4jConfigListener，官方建议使用Log4j2 
2. Spring5框架整合Log4j2

第一步 引入jar包
第二步 创建log4j2.xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->
<!--Configuration后面的status用于设置log4j2自身内部的信息输出，可以不设置，当设置成trace时，可以看到log4j2内部各种详细输出-->
<configuration status="INFO">
    <!--先定义所有的appender-->
    <appenders>
        <!--输出日志信息到控制台-->
        <console name="Console" target="SYSTEM_OUT">
            <!--控制日志输出的格式-->
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
    </console>
    </appenders>
    <!--然后定义logger，只有定义了logger并引入的appender，appender才会生效-->
    <!--root：用于指定项目的根日志，如果没有单独指定Logger，则会使用root作为默认的日志输出-->
    <loggers>
        <root level="info">
            <appender-ref ref="Console"/>
        </root>
    </loggers>
</configuration>
```

```java
public class UserLog {
    private static final Logger log = LoggerFactory.getLogger(UserLog.class);
    public static void main(String[] args) {
        log.info("hello log4j2");
        log.warn("hello log4j2");
    }
}
```

### 支持@Nullable注解

可使用在方法上面，属性上面，参数上面，表示方法返回可以为空，属性值可以为空，参数值可以为空

### 支持函数式风格GenericApplicationContext

```java
//函数式风格创建对象，交给spring进行管理
@Test
public void testGenericApplicationContext() {
    //1 创建GenericApplicationContext对象
    GenericApplicationContext context = new GenericApplicationContext();
    //2 调用context的方法对象注册
    context.refresh();
    context.registerBean("user1",User.class,() -> new User());
    //3 获取在spring注册的对象
    // User user = (User)context.getBean("com.atguigu.spring5.test.User");
    User user = (User)context.getBean("user1");
    System.out.println(user);
}
```

### JUnit5

1. 整合JUnit4
   第一步 引入Spring相关针对测试依赖

   第二步 创建测试类，使用注解方式完成

   ```java
   @RunWith(SpringJUnit4ClassRunner.class) //单元测试框架
   @ContextConfiguration("classpath:bean1.xml") //加载配置文件
   public class JTest4 {
       @Autowired
       private UserService userService;
       @Test
       public void test1() {
           userService.accountMoney();
       }
   }
   ```

2. Spring5整合JUnit5
   第一步 引入JUnit5的jar包
   第二步 创建测试类，使用注解完成

   ```java
   //@ExtendWith(SpringExtension.class)
   //@ContextConfiguration("classpath:bean1.xml")
   @SpringJUnitConfig(locations = "classpath:bean1.xml")//一个替代上面两个
   public class JTest5 {
       @Autowired
       private UserService userService;
       @Test
       public void test1() {
           userService.accountMoney();
       }
   }
   ```

   



















### Webflux

#### 1.SpringWebflux介绍

响应式编程框架



使用传统web框架，比如SpringMVC，这些基于Servlet容器，Webflux是一种异步非阻塞的框架，异步非阻塞的框架在Servlet3.1以后才支持，核心是基于Reactor的相关API实现的。



**异步非阻塞**

异步和同步针对调用者，调用者发送请求，如果等着对方回应之后才去做其他事情就是同步，如果发送请求之后不等着对方回应就去做其他事情就是异步
阻塞和非阻塞针对被调用者，被调用者受到请求之后，做完请求任务之后才给出反馈就是阻塞，受到请求之后马上给出反馈然后再去做事情就是非阻塞



**Webflux特点：**
第一 非阻塞式：在有限资源下，提高系统吞吐量和伸缩性，以Reactor为基础实现响应式编程
第二 函数式编程：Spring5框架基于java8，Webflux使用Java8函数式编程方式实现路由请求



**比较SpringMVC**
第一 两个框架都可以使用注解方式，都运行在Tomet等容器中

第二 SpringMVC采用命令式编程，Webflux采用异步响应式编程

![image-20210818154343336](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210818154343336.png)

#### 2.响应式编程（Java实现）

响应式编程是一种面向数据流和变化传播的编程范式。这意味着可以在编程语言中很方便地表达静态或动态的数据流，而相关的计算模型会自动将变化的值通过数据流进行传播。 电子表格程序就是响应式编程的一个例子。单元格可以包含字面值或类似"=B1+C1"的公式，而包含公式的单元格的值会依据其他单元格的值的变化而变化。



**Java8及其之前版本**

提供的观察者模式两个类Observer和Observable

```java
public class ObserverDemo extends Observable { 
    public static void main(String[] args) { 
        ObserverDemo observer = new ObserverDemo();  
        //添加观察者 
        observer.addObserver((o,arg)->{ System.out.println("发生变化"); }); 
        observer.addObserver((o,arg)->{ System.out.println("手动被观察者通知，准备改变"); }); 
        observer.setChanged(); //数据变化 
        observer.notifyObservers(); //通知 
    } 
}
```

java9  Flow

```java
public static void main(String[] args) {
    Flow.Publisher<String> publisher = subscriber -> {
        subscriber.onNext("1"); // 1
        subscriber.onNext("2");
        subscriber.onError(new RuntimeException("出错")); // 2
        //  subscriber.onComplete();
    };
    publisher.subscribe(new Flow.Subscriber<>() {
        @Override
        public void onSubscribe(Flow.Subscription subscription) {
            subscription.cancel();
        }
        @Override
        public void onNext(String item) {
            System.out.println(item);
        }
        @Override
        public void onError(Throwable throwable) {
            System.out.println("出错了");
        }
        @Override
        public void onComplete() {
            System.out.println("publish complete");
        }
    });
}
```

#### 3.响应式编程（Reactor实现）

1. 响应式编程操作中，Reactor是满足Reactive规范框架
2. Reactor有两个核心类，Mono和Flux，这两个类实现接口Publisher，提供丰富操作符。Flux对象实现发布者，返回N个元素；Mono实现发布者，返回0或者1个元素 
3. Flux和Mono都是数据流的发布者，使用Flux和Mono都可以发出三种数据信号： 元素值，错误信号，完成信号，错误信号和完成信号都代表终止信号，终止信号用于告诉订阅者数据流结束了，错误信号终止数据流同时把错误信息传递给订阅者

**demo**

```xml
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
    <version>3.1.5.RELEASE</version> 
</dependency>
```

```java
//just方法直接声明 
Flux.just(1,2,3,4); Mono.just(1); 
//其他的方法 
Integer[] array = {1,2,3,4}; 
Flux.fromArray(array); 

List<Integer> list = Arrays.asList(array); 
Flux.fromIterable(list); 

Stream<Integer> stream = list.stream(); 
Flux.fromStream(stream);
```

**三种信号特点**

* 错误信号和完成信号都是终止信号，不能共存的
* 如果没有发送任何元素值，而是直接发送错误或者完成信号，表示是空数据流
* 如果没有错误信号，没有完成信号，表示是无限数据流



调用just或者其他方法只是声明数据流，数据流并没有发出，只有进行订阅之后才会触发数据流，不订阅什么都不会发生的

```java
Flux.just(1234).subscribe(System.out::println);
Mono.just(123).subscribe(System.out::println);
```

**操作符**

* map 元素映射为新元素
* flatMap 元素映射为流
  把每个元素转换流，把转换之后多个流合并大的流 
* filter
* zip

#### 4.SpringWebflux执行流程和核心API

SpringWebflux基于Reactor，默认使用容器是Netty，Netty是高性能的NIO框架，异步非阻塞的框架

1. Netty

BIO阻塞

NIO非阻塞

![image-20210818170515391](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210818170515391.png)



##### SpringWebflux执行过程

与SpringMVC相似的

* SpringWebflux核心控制器 DispatchHandler，实现接口WebHandler
* 接口WebHandler有一个方法

```java
public interface WebHandler {
    Mono<Void> handle(ServerWebExchange exchange);
}
public Mono<Void> handle(ServerWebExchange exchange) {//存放http请求响应信息
    return this.handlerMappings == null ? this.createNotFoundError() : Flux.fromIterable(this.handlerMappings).concatMap((mapping) -> {
        return mapping.getHandler(exchange);//根据请求地址获取对应的mapping
    }).next().switchIfEmpty(this.createNotFoundError()).flatMap((handler) -> {
        return this.invokeHandler(exchange, handler);//调用业务方法
    }).flatMap((result) -> {
        return this.handleResult(exchange, result);//处理返回结果
    });
}
```

SpringWebflux里面DispatcherHandler，负责请求的处理
* HandlerMapping：请求查询到处理的方法
* HandlerAdapter：真正负责请求处理
* HandlerResultHandler：响应结果处理

SpringWebflux实现函数式编程，两个接口：RouterFunction（路由处理）和HandlerFunction（处理函数）

#### 5.SpringWebflux（基于注解编程模型）

SpringWebflux实现方式有两种：注解编程模型和函数式编程模型
使用注解编程模型方式，和之前SpringMVC使用相似的，只需要把相关依赖配置到项目中，SpringBoot自动配置相关运行容器，默认情况下使用Netty服务器



SpringMVC方式实现，同步阻塞的方式，基于SpringMVC+Servlet+Tomcat
SpringWebflux方式实现，异步非阻塞 方式，基于SpringWebflux+Reactor+Netty



##### demo

第一步 创建SpringBoot工程，引入Webflux依赖

```xml
spring-boot-starter-webflux
```

配置启动端口号

创建包和相关类

实体类

创建接口定义操作的方法

```java
//用户操作接口
public interface UserService {
    //根据id查询用户
    Mono<User> getUserById(int id);
    //查询所有用户
    Flux<User> getAllUser();
    //添加用户
    Mono<Void> saveUserInfo(Mono<User> user);
}
@Repository
public class UserServiceImpl implements UserService {
    //创建map集合存储数据
    private final Map<Integer,User> users = new HashMap<>();
    public UserServiceImpl() {
        this.users.put(1,new User("lucy","nan",20));
        this.users.put(2,new User("mary","nv",30));
        this.users.put(3,new User("jack","nv",50));
    }
    //根据id查询
    @Override
    public Mono<User> getUserById(int id) {
        return Mono.justOrEmpty(this.users.get(id));
    }
    //查询多个用户
    @Override
    public Flux<User> getAllUser() {
        return Flux.fromIterable(this.users.values());
    }
    //添加用户
    @Override
    public Mono<Void> saveUserInfo(Mono<User> userMono) {
        return userMono.doOnNext(person -> {
            //向map集合里面放值
            int id = users.size()+1;
            users.put(id,person);
        }).thenEmpty(Mono.empty());
    }
}
```

controller

```java
@RestController
public class UserController {
    //注入service
    @Autowired
    private UserService userService;
    //id查询
    @GetMapping("/user/{id}")
    public Mono<User> geetUserId(@PathVariable int id) {
        return userService.getUserById(id);
    }
    //查询所有
    @GetMapping("/user")
    public Flux<User> getUsers() {
        return userService.getAllUser();
    }
    //添加
    @PostMapping("/saveuser")
    public Mono<Void> saveUser(@RequestBody User user) {
        Mono<User> userMono = Mono.just(user);
        return userService.saveUserInfo(userMono);
    }
}
```

#### 6.SpringWebflux（基于函数式编程模型）

1. 在使用函数式编程模型操作时候，需要自己初始化服务器

2. 基于函数式编程模型时候，有两个核心接口：RouterFunction（实现路由功能，请求转发给对应的handler）和HandlerFunction（处理请求生成响应的函数）。核心任务定义两个函数式接口的实现并且启动需要的服务器。

3. SpringWebflux请求和响应不再是ServletRequest和ServletResponse，而是ServerRequest和ServerResponse

   

##### demo

第一步 把注解编程模型工程复制一份 ，保留entity和service内容
第二步 创建Handler（具体实现方法）

```java
import static org.springframework.web.reactive.function.BodyInserters.fromObject;

public class UserHandler {
    private final UserService userService;
    public UserHandler(UserService userService) {
        this.userService = userService;
    }
    //根据id查询
    public Mono<ServerResponse> getUserById(ServerRequest request) {
        //获取id值
        int userId = Integer.valueOf(request.pathVariable("id"));
        //空值处理
        Mono<ServerResponse> notFound = ServerResponse.notFound().build();
        //调用service方法得到数据
        Mono<User> userMono = this.userService.getUserById(userId);
        //把userMono进行转换返回
        //使用Reactor操作符flatMap
        return
            userMono
            .flatMap(person -> ServerResponse.ok().contentType(MediaType.APPLICATION_JSON)
                     .body(fromObject(person)))
            .switchIfEmpty(notFound);
    }
    //查询所有
    public Mono<ServerResponse> getAllUsers(ServerRequest request) {
        //调用service得到结果
        Flux<User> users = this.userService.getAllUser();
        return ServerResponse.ok().contentType(MediaType.APPLICATION_JSON)
            .body(users,User.class);
    }
    //添加
    public Mono<ServerResponse> saveUser(ServerRequest request) {
        //得到user对象
        Mono<User> userMono = request.bodyToMono(User.class);
        return ServerResponse.ok().build(this.userService.saveUserInfo(userMono));
    }
}
```

第三步 初始化服务器，编写Router

* 创建路由的方法

```java
import static org.springframework.http.MediaType.APPLICATION_JSON;
import static org.springframework.web.reactive.function.server.RequestPredicates.GET;
import static org.springframework.web.reactive.function.server.RequestPredicates.accept;
import static org.springframework.web.reactive.function.server.RouterFunctions.toHttpHandler;

public class Server {//最终调用,浏览器调用
    public static void main(String[] args) throws Exception{
        Server server = new Server();
        server.createReactorServer();
        System.out.println("enter to exit");
        System.in.read();
    }
    //1 创建Router路由
    public RouterFunction<ServerResponse> routingFunction() {
        //创建hanler对象
        UserService userService = new UserServiceImpl();
        UserHandler handler = new UserHandler(userService);
        //设置路由
        return RouterFunctions.route(
                GET("/users/{id}").and(accept(APPLICATION_JSON)),handler::getUserById)
                .andRoute(GET("/users").and(accept(APPLICATION_JSON)),handler::getAllUsers);
    }
    //2 创建服务器完成适配
    public void createReactorServer() {
        //路由和handler适配
        RouterFunction<ServerResponse> route = routingFunction();
        HttpHandler httpHandler = toHttpHandler(route);
        ReactorHttpHandlerAdapter adapter = new ReactorHttpHandlerAdapter(httpHandler);
        //创建服务器
        HttpServer httpServer = HttpServer.create();
        httpServer.handle(adapter).bindNow();
    }
}
```

client调用

```java
public class Client {

    public static void main(String[] args) {
        //调用服务器地址
        WebClient webClient = WebClient.create("http://127.0.0.1:5794");

        //根据id查询
        String id = "1";
        User userresult = webClient.get().uri("/users/{id}", id)
                .accept(MediaType.APPLICATION_JSON).retrieve().bodyToMono(User.class)
                .block();
        System.out.println(userresult.getName());

        //查询所有
        Flux<User> results = webClient.get().uri("/users")
                .accept(MediaType.APPLICATION_JSON).retrieve().bodyToFlux(User.class);

        results.map(stu -> stu.getName())
                    .buffer().doOnNext(System.out::println).blockFirst();
    }
}
```

# SpringMVC

## SpringMVC简介

### MVC

MVC是一种软件架构的思想，将软件按照模型、视图、控制器来划分

M：Model，模型层，指工程中的JavaBean，作用是处理数据

JavaBean分为两类：

- 一类称为实体类Bean：专门存储业务数据的，如 Student、User 等
- 一类称为业务处理 Bean：指 Service 或 Dao 对象，专门用于处理业务逻辑和数据访问。

V：View，视图层，指工程中的html或jsp等页面，作用是与用户进行交互，展示数据

C：Controller，控制层，指工程中的servlet，作用是接收请求和响应浏览器

MVC的工作流程：
用户通过视图层发送请求到服务器，在服务器中请求被Controller接收，Controller调用相应的Model层处理请求，处理完毕将结果返回到Controller，Controller再根据请求处理的结果找到相应的View视图，渲染数据后最终响应给浏览器

### SpringMVC

SpringMVC 是 Spring 为表述层开发提供的一整套完备的解决方案

### 特点

- **Spring 家族原生产品**，与 IOC 容器等基础设施无缝对接
- **基于原生的Servlet**，通过了功能强大的**前端控制器DispatcherServlet**，对请求和响应进行统一处理
- 表述层各细分领域需要解决的问题**全方位覆盖**，提供**全面解决方案**
- **代码清新简洁**，大幅度提升开发效率
- 内部组件化程度高，可插拔式组件**即插即用**，想要什么功能配置相应组件即可
- **性能卓著**，尤其适合现代大型、超大型互联网项目要求

## HelloWorld

### 创建maven工程

```xml
<dependencies>
    <!-- SpringMVC -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.3.1</version>
    </dependency>

    <!-- 日志 -->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.2.3</version>
    </dependency>

    <!-- ServletAPI -->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
        <scope>provided</scope>
    </dependency>

    <!-- Spring5和Thymeleaf整合包 -->
    <dependency>
        <groupId>org.thymeleaf</groupId>
        <artifactId>thymeleaf-spring5</artifactId>
        <version>3.0.12.RELEASE</version>
    </dependency>
</dependencies>
```

由于 Maven 的传递性，我们不必将所有需要的包全部配置依赖，而是配置最顶端的依赖，其他靠传递性导入。

### 配置web.xml

注册SpringMVC的前端控制器DispatcherServlet

##### 默认配置方式

此配置作用下，SpringMVC的配置文件默认位于WEB-INF下，默认名称为\<servlet-name>-servlet.xml，例如，以下配置所对应SpringMVC的配置文件位于WEB-INF下，文件名为springMVC-servlet.xml

```xml
<!-- 配置SpringMVC的前端控制器，对浏览器发送的请求统一进行处理 -->
<servlet>
    <servlet-name>springMVC</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>springMVC</servlet-name>
    <!--
        设置springMVC的核心控制器所能处理的请求的请求路径
        /所匹配的请求可以是/login或.html或.js或.css方式的请求路径
        但是/不能匹配.jsp请求路径的请求
    -->
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

##### 扩展配置方式

可通过init-param标签设置SpringMVC配置文件的位置和名称，通过load-on-startup标签设置SpringMVC前端控制器DispatcherServlet的初始化时间

```xml
<!-- 配置SpringMVC的前端控制器，对浏览器发送的请求统一进行处理 -->
<servlet>
    <servlet-name>springMVC</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- 通过初始化参数指定SpringMVC配置文件的位置和名称 -->
    <init-param>
        <!-- contextConfigLocation为固定值 -->
        <param-name>contextConfigLocation</param-name>
        <!-- 使用classpath:表示从类路径查找配置文件，例如maven工程中的src/main/resources -->
        <param-value>classpath:springMVC.xml</param-value>
    </init-param>
    <!-- 
 		作为框架的核心组件，在启动过程中有大量的初始化操作要做
		而这些操作放在第一次请求时才执行会严重影响访问速度
		因此需要通过此标签将启动控制DispatcherServlet的初始化时间提前到服务器启动时
	-->
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>springMVC</servlet-name>
    <!--
        设置springMVC的核心控制器所能处理的请求的请求路径
        /所匹配的请求可以是/login或.html或.js或.css方式的请求路径
        但是/不能匹配.jsp请求路径的请求
    -->
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

> 注：
>
> \<url-pattern>标签中使用/和/*的区别：
>
> /所匹配的请求可以是/login或.html或.js或.css方式的请求路径，但是/不能匹配.jsp请求路径的请求
>
> 因此就可以避免在访问jsp页面时，该请求被DispatcherServlet处理，从而找不到相应的页面
>
> /*则能够匹配所有请求，例如在使用过滤器时，若需要对所有请求进行过滤，就需要使用/\*的写法

### 创建请求控制器

```java
@Controller
public class HelloController {
    
}
```

### 创建springMVC的配置文件

```xml
<!-- 自动扫描包 -->
<context:component-scan base-package="com.atguigu.mvc.controller"/>

<!-- 配置Thymeleaf视图解析器 -->
<bean id="viewResolver" class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
    <property name="order" value="1"/>
    <property name="characterEncoding" value="UTF-8"/>
    <property name="templateEngine">
        <bean class="org.thymeleaf.spring5.SpringTemplateEngine">
            <property name="templateResolver">
                <bean class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">
    
                    <!-- 视图前缀 -->
                    <property name="prefix" value="/WEB-INF/templates/"/>
    
                    <!-- 视图后缀 -->
                    <property name="suffix" value=".html"/>
                    <property name="templateMode" value="HTML5"/>
                    <property name="characterEncoding" value="UTF-8" />
                </bean>
            </property>
        </bean>
    </property>
</bean>

<!-- 
   处理静态资源，例如html、js、css、jpg
  若只设置该标签，则只能访问静态资源，其他请求则无法访问
  此时必须设置<mvc:annotation-driven/>解决问题
 -->
<mvc:default-servlet-handler/>

<!-- 开启mvc注解驱动 -->
<mvc:annotation-driven>
    <mvc:message-converters>
        <!-- 处理响应中文内容乱码 -->
        <bean class="org.springframework.http.converter.StringHttpMessageConverter">
            <property name="defaultCharset" value="UTF-8" />
            <property name="supportedMediaTypes">
                <list>
                    <value>text/html</value>
                    <value>application/json</value>
                </list>
            </property>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>
```

### 测试HelloWorld

##### a > 实现对首页的访问

在请求控制器中创建处理请求的方法

```java
// @RequestMapping注解：处理请求和控制器方法之间的映射关系
// @RequestMapping注解的value属性可以通过请求地址匹配请求，/表示的当前工程的上下文路径
// localhost:8080/springMVC/
@RequestMapping("/")
public String index() {
    //设置视图名称
    return "index";
}
```

##### b>通过超链接跳转到指定页面

在主页index.html中设置超链接

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>首页</title>
</head>
<body>
    <h1>首页</h1>
    <a th:href="@{/hello}">HelloWorld</a><br/>
</body>
</html>
```

在请求控制器中创建处理请求的方法

```java
@RequestMapping("/hello")
public String HelloWorld() {
    return "target";
}
```

### 总结

浏览器发送请求，若请求地址符合前端控制器的url-pattern，该请求就会被前端控制器DispatcherServlet处理。前端控制器会读取SpringMVC的核心配置文件，通过扫描组件找到控制器，将请求地址和控制器中@RequestMapping注解的value属性值进行匹配，若匹配成功，该注解所标识的控制器方法就是处理请求的方法。处理请求的方法需要返回一个字符串类型的视图名称，该视图名称会被视图解析器解析，加上前缀和后缀组成视图的路径，通过Thymeleaf对视图进行渲染，最终转发到视图所对应页面

## @RequestMapping

```java
@RequestMapping(
        value = {"/testRequestMapping", "/test"},
        method = {RequestMethod.GET, RequestMethod.POST}
)
public String testRequestMapping(){
    return "success";
}
```

### params属性

请求参数匹配请求映射

params属性是一个字符串类型的数组，可以通过四种表达式设置请求参数和请求映射的匹配关系

```
"param"：要求请求映射所匹配的请求必须携带param请求参数

"!param"：要求请求映射所匹配的请求必须不能携带param请求参数

"param=value"：要求请求映射所匹配的请求必须携带param请求参数且param=value

"param!=value"：要求请求映射所匹配的请求必须携带param请求参数但是param!=value
```

```html
<a th:href="@{/test(username='admin',password=123456)">测试@RequestMapping的params属性-->/test</a><br>
```

```java
@RequestMapping(
        value = {"/testRequestMapping", "/test"}
        ,method = {RequestMethod.GET, RequestMethod.POST}
        ,params = {"username","password!=123456"}
)
public String testRequestMapping(){
    return "success";
}
```

> 注：
>
> 若当前请求满足@RequestMapping注解的value和method属性，但是不满足params属性，此时页面回报错400：Parameter conditions "username, password!=123456" not met for actual request parameters: username={admin}, password={123456}

### headers属性

请求头信息匹配请求映射

headers属性是一个字符串类型的数组，可以通过四种表达式设置请求头信息和请求映射的匹配关系

```
"header"：要求请求映射所匹配的请求必须携带header请求头信息

"!header"：要求请求映射所匹配的请求必须不能携带header请求头信息

"header=value"：要求请求映射所匹配的请求必须携带header请求头信息且header=value

"header!=value"：要求请求映射所匹配的请求必须携带header请求头信息且header!=value
```

若当前请求满足@RequestMapping注解的value和method属性，但是不满足headers属性，此时页面显示404错误，即资源未找到

### ant风格的路径

？：表示任意的单个字符

*：表示任意的0个或多个字符

\**：表示任意的一层或多层目录

注意：在使用\**时，只能使用/**/xxx的方式

## 获取请求参数

### 通过ServletAPI获取

将HttpServletRequest作为控制器方法的形参，此时HttpServletRequest类型的参数表示封装了当前请求的请求报文的对象



```java
@RequestMapping("/testParam")
public String testParam(HttpServletRequest request){
    String username = request.getParameter("username");
    String password = request.getParameter("password");
    System.out.println("username:"+username+",password:"+password);
    return "success";
}
```

### 控制器方法的形参获取请求参数

在控制器方法的形参位置，设置和请求参数同名的形参，当浏览器发送请求，匹配到请求映射时，在DispatcherServlet中就会将请求参数赋值给相应的形参

```html
<a th:href="@{/testParam(username='admin',password=123456)}">测试获取请求参数-->/testParam</a><br>
```

```java
@RequestMapping("/testParam")
public String testParam(String username, String password){
    System.out.println("username:"+username+",password:"+password);
    return "success";
}
```

> 注：
>
> 若请求所传输的请求参数中有多个同名的请求参数，此时可以在控制器方法的形参中设置字符串数组或者字符串类型的形参接收此请求参数
>
> 若使用字符串数组类型的形参，此参数的数组中包含了每一个数据
>
> 若使用字符串类型的形参，此参数的值为每个数据中间使用逗号拼接的结果

### @RequestParam/@RequestHeader/@CookieValue

@RequestParam是将请求参数和控制器方法的形参创建映射关系

@RequestParam注解一共有三个属性：

value：指定为形参赋值的请求参数的参数名

required：设置是否必须传输此请求参数，默认值为true

若设置为true时，则当前请求必须传输value所指定的请求参数，若没有传输该请求参数，且没有设置defaultValue属性，则页面报错400：Required String parameter 'xxx' is not present；若设置为false，则当前请求不是必须传输value所指定的请求参数，若没有传输，则注解所标识的形参的值为null

defaultValue：不管required属性值为true或false，当value所指定的请求参数没有传输或传输的值为""时，则使用默认值为形参赋值



@RequestHeader是将请求头信息和控制器方法的形参创建映射关系

@RequestHeader注解一共有三个属性：value、required、defaultValue，用法同@RequestParam



@CookieValue是将cookie数据和控制器方法的形参创建映射关系

@CookieValue注解一共有三个属性：value、required、defaultValue，用法同@RequestParam

### POJO获取请求参数

可以在控制器方法的形参位置设置一个实体类类型的形参，此时若浏览器传输的请求参数的参数名和实体类中的属性名一致，那么请求参数就会为此属性赋值

```html
<form th:action="@{/testpojo}" method="post">
    用户名：<input type="text" name="username"><br>
    密码：<input type="password" name="password"><br>
    性别：<input type="radio" name="sex" value="男">男<input type="radio" name="sex" value="女">女<br>
    年龄：<input type="text" name="age"><br>
    邮箱：<input type="text" name="email"><br>
    <input type="submit">
</form>
```

```java
@RequestMapping("/testpojo")
public String testPOJO(User user){
    System.out.println(user);
    return "success";
}
//最终结果-->User{id=null, username='张三', password='123', age=23, sex='男', email='123@qq.com'}
```

### 解决请求参数的乱码问题

解决获取请求参数的乱码问题，可以使用SpringMVC提供的编码过滤器CharacterEncodingFilter，但是必须在web.xml中进行注册

```xml
<!--配置springMVC的编码过滤器-->
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
        <param-name>forceResponseEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

> 注：
>
> SpringMVC中处理编码的过滤器一定要配置到其他过滤器之前，否则无效

## 域对象共享数据

session浏览器

servletcontext服务器

request一次请求

### 1、使用ServletAPI向request域对象共享数据

```java
@RequestMapping("/testServletAPI")
public String testServletAPI(HttpServletRequest request){
    request.setAttribute("testScope", "hello,servletAPI");
    return "success";
}
```

### 2、使用ModelAndView向request域对象共享数据

```java
@RequestMapping("/testModelAndView")
public ModelAndView testModelAndView(){
    /**
     * ModelAndView有Model和View的功能
     * Model主要用于向请求域共享数据
     * View主要用于设置视图，实现页面跳转
     */
    ModelAndView mav = new ModelAndView();
    //向请求域共享数据
    mav.addObject("testScope", "hello,ModelAndView");
    //设置视图，实现页面跳转
    mav.setViewName("success");
    return mav;
}
```

### 3、使用Model向request域对象共享数据

```java
@RequestMapping("/testModel")
public String testModel(Model model){
    model.addAttribute("testScope", "hello,Model");
    return "success";
}
```

### 4、使用map向request域对象共享数据

```java
@RequestMapping("/testMap")
public String testMap(Map<String, Object> map){
    map.put("testScope", "hello,Map");
    return "success";
}
```

### 5、使用ModelMap向request域对象共享数据

```java
@RequestMapping("/testModelMap")
public String testModelMap(ModelMap modelMap){
    modelMap.addAttribute("testScope", "hello,ModelMap");
    return "success";
}
```

### 6、Model、ModelMap、Map的关系

Model、ModelMap、Map类型的参数其实本质上都是 BindingAwareModelMap 类型的

```java
public interface Model{}
public class ModelMap extends LinkedHashMap<String, Object> {}
public class ExtendedModelMap extends ModelMap implements Model {}
public class BindingAwareModelMap extends ExtendedModelMap {}
```

### 7、向session域共享数据

```java
@RequestMapping("/testSession")
public String testSession(HttpSession session){
    session.setAttribute("testSessionScope", "hello,session");
    return "success";
}
```

### 8、向application域共享数据

```java
@RequestMapping("/testApplication")
public String testApplication(HttpSession session){
	ServletContext application = session.getServletContext();
    application.setAttribute("testApplicationScope", "hello,application");
    return "success";
}
```

## SpringMVC的视图

SpringMVC中的视图是View接口，视图的作用渲染数据，将模型Model中的数据展示给用户

SpringMVC视图的种类很多，默认有转发视图和重定向视图

当工程引入jstl的依赖，转发视图会自动转换为JstlView

若使用的视图技术为Thymeleaf，在SpringMVC的配置文件中配置了Thymeleaf的视图解析器，由此视图解析器解析之后所得到的是ThymeleafView

### ThymeleafView

当控制器方法中所设置的视图名称没有任何前缀时，此时的视图名称会被SpringMVC配置文件中所配置的视图解析器解析，视图名称拼接视图前缀和视图后缀所得到的最终路径，会通过转发的方式实现跳转

```java
@RequestMapping("/testHello")
public String testHello(){
    return "hello";
}
```

### 转发视图

SpringMVC中默认的转发视图是InternalResourceView

SpringMVC中创建转发视图的情况：

当控制器方法中所设置的视图名称以"forward:"为前缀时，创建InternalResourceView视图，此时的视图名称不会被SpringMVC配置文件中所配置的视图解析器解析，而是会将前缀"forward:"去掉，剩余部分作为最终路径通过转发的方式实现跳转

例如"forward:/"，"forward:/employee"

```java
@RequestMapping("/testForward")
public String testForward(){
    return "forward:/testHello";
}
```

### 重定向视图

SpringMVC中默认的重定向视图是RedirectView

当控制器方法中所设置的视图名称以"redirect:"为前缀时，创建RedirectView视图，此时的视图名称不会被SpringMVC配置文件中所配置的视图解析器解析，而是会将前缀"redirect:"去掉，剩余部分作为最终路径通过重定向的方式实现跳转

例如"redirect:/"，"redirect:/employee"

```java
@RequestMapping("/testRedirect")
public String testRedirect(){
    return "redirect:/testHello";
}
```

> 注
>
> 重定向视图在解析时，会先将redirect:前缀去掉，然后会判断剩余部分是否以/开头，若是则会自动拼接上下文路径

### 视图控制器view-controller

当控制器方法中，仅仅用来实现页面跳转，即只需要设置视图名称时，可以将处理器方法使用view-controller标签进行表示

```xml
<!--
	path：设置处理的请求地址
	view-name：设置请求地址所对应的视图名称
-->
<mvc:view-controller path="/testView" view-name="success"></mvc:view-controller>
```

> 注：
>
> 当SpringMVC中设置任何一个view-controller时，其他控制器中的请求映射将全部失效，此时需要在SpringMVC的核心配置文件中设置开启mvc注解驱动的标签：
>
> <mvc:annotation-driven />

## RESTful

**Re**presentational **S**tate **T**ransfer，表现层资源状态转移

GET、POST、PUT、DELETE

### HiddenHttpMethodFilter

由于浏览器只支持发送get和post方式的请求，那么该如何发送put和delete请求呢？

SpringMVC 提供了 **HiddenHttpMethodFilter** 帮助我们**将 POST 请求转换为 DELETE 或 PUT 请求**

**HiddenHttpMethodFilter** 处理put和delete请求的条件：

a>当前请求的请求方式必须为post

b>当前请求必须传输请求参数_method

满足以上条件，**HiddenHttpMethodFilter** 过滤器就会将当前请求的请求方式转换为请求参数_method的值，因此请求参数\_method的值才是最终的请求方式

在web.xml中注册**HiddenHttpMethodFilter** 

```xml
<filter>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

> 注：
>
> 目前为止，SpringMVC中提供了两个过滤器：CharacterEncodingFilter和HiddenHttpMethodFilter
>
> 在web.xml中注册时，必须先注册CharacterEncodingFilter，再注册HiddenHttpMethodFilter
>
> 原因：
>
> - 在 CharacterEncodingFilter 中通过 request.setCharacterEncoding(encoding) 方法设置字符集的
>
> - request.setCharacterEncoding(encoding) 方法要求前面不能有任何获取请求参数的操作
>
> - 而 HiddenHttpMethodFilter 恰恰有一个获取请求方式的操作：
>
> - ```
>   String paramValue = request.getParameter(this.methodParam);
>   ```

### RESTful案例

#### GET

```java
@RequestMapping(value = "/employee", method = RequestMethod.GET)
public String getEmployeeList(Model model){
    Collection<Employee> employeeList = employeeDao.getAll();
    model.addAttribute("employeeList", employeeList);
    return "employee_list";
}
```

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Employee Info</title>
    <script type="text/javascript" th:src="@{/static/js/vue.js}"></script>
</head>
<body>

    <table border="1" cellpadding="0" cellspacing="0" style="text-align: center;" id="dataTable">
        <tr>
            <th colspan="5">Employee Info</th>
        </tr>
        <tr>
            <th>id</th>
            <th>lastName</th>
            <th>email</th>
            <th>gender</th>
            <th>options(<a th:href="@{/toAdd}">add</a>)</th>
        </tr>
        <tr th:each="employee : ${employeeList}">
            <td th:text="${employee.id}"></td>
            <td th:text="${employee.lastName}"></td>
            <td th:text="${employee.email}"></td>
            <td th:text="${employee.gender}"></td>
            <td>
                <a class="deleteA" @click="deleteEmployee" th:href="@{'/employee/'+${employee.id}}">delete</a>
                <a th:href="@{'/employee/'+${employee.id}}">update</a>
            </td>
        </tr>
    </table>
</body>
</html>
```

#### DELETE

##### a>创建处理delete请求方式的表单

```html
<!-- 作用：通过超链接控制表单的提交，将post请求转换为delete请求 -->
<form id="delete_form" method="post">
    <!-- HiddenHttpMethodFilter要求：必须传输_method请求参数，并且值为最终的请求方式 -->
    <input type="hidden" name="_method" value="delete"/>
</form>
```

##### b>删除超链接绑定点击事件

引入vue.js

```html
<script type="text/javascript" th:src="@{/static/js/vue.js}"></script>
```

删除超链接

```html
<a class="deleteA" @click="deleteEmployee" th:href="@{'/employee/'+${employee.id}}">delete</a>
```

通过vue处理点击事件

```html
<script type="text/javascript">
    var vue = new Vue({
        el:"#dataTable",
        methods:{
            //event表示当前事件
            deleteEmployee:function (event) {
                //通过id获取表单标签
                var delete_form = document.getElementById("delete_form");
                //将触发事件的超链接的href属性为表单的action属性赋值
                delete_form.action = event.target.href;
                //提交表单
                delete_form.submit();
                //阻止超链接的默认跳转行为
                event.preventDefault();
            }
        }
    });
</script>
```

##### c>控制器方法

```java
@RequestMapping(value = "/employee/{id}", method = RequestMethod.DELETE)
public String deleteEmployee(@PathVariable("id") Integer id){
    employeeDao.delete(id);
    return "redirect:/employee";
}
```

#### POST

##### a>配置view-controller

```xml
<mvc:view-controller path="/toAdd" view-name="employee_add"></mvc:view-controller>
```

##### b>创建employee_add.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Add Employee</title>
</head>
<body>

<form th:action="@{/employee}" method="post">
    lastName:<input type="text" name="lastName"><br>
    email:<input type="text" name="email"><br>
    gender:<input type="radio" name="gender" value="1">male
    <input type="radio" name="gender" value="0">female<br>
    <input type="submit" value="add"><br>
</form>

</body>
</html>
```

##### c>控制器方法

```java
@RequestMapping(value = "/employee", method = RequestMethod.POST)
public String addEmployee(Employee employee){
    employeeDao.save(employee);
    return "redirect:/employee";
}
```

#### PUT

##### a>修改超链接

```html
<a th:href="@{'/employee/'+${employee.id}}">update</a>
```

##### b>控制器方法

```java
@RequestMapping(value = "/employee/{id}", method = RequestMethod.GET)
public String getEmployeeById(@PathVariable("id") Integer id, Model model){
    Employee employee = employeeDao.get(id);
    model.addAttribute("employee", employee);
    return "employee_update";
}
```

##### c>创建employee_update.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Update Employee</title>
</head>
<body>

<form th:action="@{/employee}" method="post">
    <input type="hidden" name="_method" value="put">
    <input type="hidden" name="id" th:value="${employee.id}">
    lastName:<input type="text" name="lastName" th:value="${employee.lastName}"><br>
    email:<input type="text" name="email" th:value="${employee.email}"><br>
    <!--
        th:field="${employee.gender}"可用于单选框或复选框的回显
        若单选框的value和employee.gender的值一致，则添加checked="checked"属性
    -->
    gender:<input type="radio" name="gender" value="1" th:field="${employee.gender}">male
    <input type="radio" name="gender" value="0" th:field="${employee.gender}">female<br>
    <input type="submit" value="update"><br>
</form>

</body>
</html>
```

##### d>控制器方法

```java
@RequestMapping(value = "/employee", method = RequestMethod.PUT)
public String updateEmployee(Employee employee){
    employeeDao.save(employee);
    return "redirect:/employee";
}
```

## HttpMessageConverter

报文信息转换器，将请求报文转换为Java对象，或将Java对象转换为响应报文

HttpMessageConverter提供了两个注解和两个类型：

@RequestBody，@ResponseBody，RequestEntity，ResponseEntity

### @RequestBody/@ResponseBody/@RestController

```java
@RequestMapping("/testRequestBody")
public String testRequestBody(@RequestBody String requestBody){
    System.out.println("requestBody:"+requestBody);
    return "success";
}
@RequestMapping("/testResponseBody")
@ResponseBody
public String testResponseBody(){
    return "success";
}
```

#### @ResponseBody处理json

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.12.1</version>
</dependency>
```

b>在SpringMVC的核心配置文件中开启mvc的注解驱动，此时在HandlerAdaptor中会自动装配一个消息转换器：MappingJackson2HttpMessageConverter，可以将响应到浏览器的Java对象转换为Json格式的字符串

```xml
<mvc:annotation-driven />
```

c>在处理器方法上使用@ResponseBody注解进行标识

d>将Java对象直接作为控制器方法的返回值返回，就会自动转换为Json格式的字符串

```java
@RequestMapping("/testResponseUser")
@ResponseBody
public User testResponseUser(){
    return new User(1001,"admin","123456",23,"男");
}
```

### RequestEntity

RequestEntity封装请求报文的一种类型，需要在控制器方法的形参中设置该类型的形参，当前请求的请求报文就会赋值给该形参，可以通过getHeaders()获取请求头信息，通过getBody()获取请求体信息

```java
@RequestMapping("/testRequestEntity")
public String testRequestEntity(RequestEntity<String> requestEntity){
    System.out.println("requestHeader:"+requestEntity.getHeaders());
    System.out.println("requestBody:"+requestEntity.getBody());
    return "success";
}
```

### 处理ajax

a>请求超链接：

```html
<div id="app">
	<a th:href="@{/testAjax}" @click="testAjax">testAjax</a><br>
</div>
```

b>通过vue和axios处理点击事件：

```html
<script type="text/javascript" th:src="@{/static/js/vue.js}"></script>
<script type="text/javascript" th:src="@{/static/js/axios.min.js}"></script>
<script type="text/javascript">
    var vue = new Vue({
        el:"#app",
        methods:{
            testAjax:function (event) {
                axios({
                    method:"post",
                    url:event.target.href,
                    params:{
                        username:"admin",
                        password:"123456"
                    }
                }).then(function (response) {
                    alert(response.data);
                });
                event.preventDefault();
            }
        }
    });
</script>
```

c>控制器方法：

```java
@RequestMapping("/testAjax")
@ResponseBody
public String testAjax(String username, String password){
    System.out.println("username:"+username+",password:"+password);
    return "hello,ajax";
}
```

### ResponseEntity

用于控制器方法的返回值类型，该控制器方法的返回值就是响应到浏览器的响应报文

#### 文件下载

```java
@RequestMapping("/testDown")
public ResponseEntity<byte[]> testResponseEntity(HttpSession session) throws IOException {
    //获取ServletContext对象
    ServletContext servletContext = session.getServletContext();
    //获取服务器中文件的真实路径
    String realPath = servletContext.getRealPath("/static/img/1.jpg");
    //创建输入流
    InputStream is = new FileInputStream(realPath);
    //创建字节数组
    byte[] bytes = new byte[is.available()];
    //将流读到字节数组中
    is.read(bytes);
    //创建HttpHeaders对象设置响应头信息
    MultiValueMap<String, String> headers = new HttpHeaders();
    //设置要下载方式以及下载文件的名字
    headers.add("Content-Disposition", "attachment;filename=1.jpg");
    //设置响应状态码
    HttpStatus statusCode = HttpStatus.OK;
    //创建ResponseEntity对象
    ResponseEntity<byte[]> responseEntity = new ResponseEntity<>(bytes, headers, statusCode);
    //关闭输入流
    is.close();
    return responseEntity;
}
```

#### 文件上传

文件上传要求form表单的请求方式必须为post，并且添加属性enctype="multipart/form-data"

SpringMVC中将上传的文件封装到MultipartFile对象中，通过此对象可以获取文件相关信息

上传步骤：

a>添加依赖：

```xml
<!-- https://mvnrepository.com/artifact/commons-fileupload/commons-fileupload -->
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.1</version>
</dependency>
```

b>在SpringMVC的配置文件中添加配置：

```xml
<!--必须通过文件解析器的解析才能将文件转换为MultipartFile对象-->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"></bean>
```

c>控制器方法：

```java
@RequestMapping("/testUp")
public String testUp(MultipartFile photo, HttpSession session) throws IOException {
    //获取上传的文件的文件名
    String fileName = photo.getOriginalFilename();
    //处理文件重名问题
    String hzName = fileName.substring(fileName.lastIndexOf("."));
    fileName = UUID.randomUUID().toString() + hzName;
    //获取服务器中photo目录的路径
    ServletContext servletContext = session.getServletContext();
    String photoPath = servletContext.getRealPath("photo");
    File file = new File(photoPath);
    if(!file.exists()){
        file.mkdir();
    }
    String finalPath = photoPath + File.separator + fileName;
    //实现上传功能
    photo.transferTo(new File(finalPath));
    return "success";
}
```

## 拦截器

### 配置

实现HandlerInterceptor

```xml
<bean class="com.atguigu.interceptor.FirstInterceptor"></bean>
<ref bean="firstInterceptor"></ref>
<!-- 以上两种配置方式都是对DispatcherServlet所处理的所有的请求进行拦截 -->
<mvc:interceptor>
    <mvc:mapping path="/**"/>
    <mvc:exclude-mapping path="/testRequestEntity"/>
    <ref bean="firstInterceptor"></ref>
</mvc:interceptor>
<!-- 
	以上配置方式可以通过ref或bean标签设置拦截器，通过mvc:mapping设置需要拦截的请求，通过mvc:exclude-mapping设置需要排除的请求，即不需要拦截的请求
-->
```

preHandle：控制器方法执行之前

postHandle：控制器方法执行之后

afterComplation：处理完视图和模型数据，渲染视图完毕之后

## 异常处理器

### 基于配置的异常处理

SpringMVC提供了一个处理控制器方法执行过程中所出现的异常的接口：HandlerExceptionResolver

HandlerExceptionResolver接口的实现类有：DefaultHandlerExceptionResolver和SimpleMappingExceptionResolver

SpringMVC提供了自定义的异常处理器SimpleMappingExceptionResolver，使用方式：

```xml
<bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
    <property name="exceptionMappings">
        <props>
        	<!--
        		properties的键表示处理器方法执行过程中出现的异常
        		properties的值表示若出现指定异常时，设置一个新的视图名称，跳转到指定页面
        	-->
            <prop key="java.lang.ArithmeticException">error</prop>
        </props>
    </property>
    <!--
    	exceptionAttribute属性设置一个属性名，将出现的异常信息在请求域中进行共享
    -->
    <property name="exceptionAttribute" value="ex"></property>
</bean>
```

### 基于注解的异常处理

```java
//@ControllerAdvice将当前类标识为异常处理的组件
@ControllerAdvice
public class ExceptionController {

    //@ExceptionHandler用于设置所标识方法处理的异常
    @ExceptionHandler(ArithmeticException.class)
    //ex表示当前请求处理中出现的异常对象
    public String handleArithmeticException(Exception ex, Model model){
        model.addAttribute("ex", ex);
        return "error";
    }

}
```

## 注解配置

### 创建初始化类，代替web.xml

在Servlet3.0环境中，容器会在类路径中查找实现javax.servlet.ServletContainerInitializer接口的类，如果找到的话就用它来配置Servlet容器。
Spring提供了这个接口的实现，名为SpringServletContainerInitializer，这个类反过来又会查找实现WebApplicationInitializer的类并将配置的任务交给它们来完成。Spring3.2引入了一个便利的WebApplicationInitializer基础实现，名为AbstractAnnotationConfigDispatcherServletInitializer，当我们的类扩展了AbstractAnnotationConfigDispatcherServletInitializer并将其部署到Servlet3.0容器的时候，容器会自动发现它，并用它来配置Servlet上下文。

```java
public class WebInit extends AbstractAnnotationConfigDispatcherServletInitializer {

    /**
     * 指定spring的配置类
     * @return
     */
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{SpringConfig.class};
    }

    /**
     * 指定SpringMVC的配置类
     * @return
     */
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{WebConfig.class};
    }

    /**
     * 指定DispatcherServlet的映射规则，即url-pattern
     * @return
     */
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

    /**
     * 添加过滤器
     * @return
     */
    @Override
    protected Filter[] getServletFilters() {
        CharacterEncodingFilter encodingFilter = new CharacterEncodingFilter();
        encodingFilter.setEncoding("UTF-8");
        encodingFilter.setForceRequestEncoding(true);
        HiddenHttpMethodFilter hiddenHttpMethodFilter = new HiddenHttpMethodFilter();
        return new Filter[]{encodingFilter, hiddenHttpMethodFilter};
    }
}
```

### SpringConfig配置类

```java
@Configuration
public class SpringConfig {
	//ssm整合之后，spring的配置信息写在此类中
}
```

### WebConfig配置类

```java
@Configuration
//扫描组件
@ComponentScan("com.atguigu.mvc.controller")
//开启MVC注解驱动
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    //使用默认的servlet处理静态资源
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    //配置文件上传解析器
    @Bean
    public CommonsMultipartResolver multipartResolver(){
        return new CommonsMultipartResolver();
    }

    //配置拦截器
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        FirstInterceptor firstInterceptor = new FirstInterceptor();
        registry.addInterceptor(firstInterceptor).addPathPatterns("/**");
    }
    
    //配置视图控制
    
    /*@Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("index");
    }*/
    
    //配置异常映射
    /*@Override
    public void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        SimpleMappingExceptionResolver exceptionResolver = new SimpleMappingExceptionResolver();
        Properties prop = new Properties();
        prop.setProperty("java.lang.ArithmeticException", "error");
        //设置异常映射
        exceptionResolver.setExceptionMappings(prop);
        //设置共享异常信息的键
        exceptionResolver.setExceptionAttribute("ex");
        resolvers.add(exceptionResolver);
    }*/

    //配置生成模板解析器
    @Bean
    public ITemplateResolver templateResolver() {
        WebApplicationContext webApplicationContext = 
            ContextLoader.getCurrentWebApplicationContext();
        // ServletContextTemplateResolver需要一个ServletContext作为构造参数，
        // 可通过WebApplicationContext 的方法获得
        ServletContextTemplateResolver templateResolver = new ServletContextTemplateResolver(
                webApplicationContext.getServletContext());
        templateResolver.setPrefix("/WEB-INF/templates/");
        templateResolver.setSuffix(".html");
        templateResolver.setCharacterEncoding("UTF-8");
        templateResolver.setTemplateMode(TemplateMode.HTML);
        return templateResolver;
    }

    //生成模板引擎并为模板引擎注入模板解析器
    @Bean
    public SpringTemplateEngine templateEngine(ITemplateResolver templateResolver) {
        SpringTemplateEngine templateEngine = new SpringTemplateEngine();
        templateEngine.setTemplateResolver(templateResolver);
        return templateEngine;
    }

    //生成视图解析器并未解析器注入模板引擎
    @Bean
    public ViewResolver viewResolver(SpringTemplateEngine templateEngine) {
        ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
        viewResolver.setCharacterEncoding("UTF-8");
        viewResolver.setTemplateEngine(templateEngine);
        return viewResolver;
    }


}
```

## SpringMVC执行流程

### SpringMVC常用组件

- DispatcherServlet：**前端控制器**，不需要工程师开发，由框架提供

作用：统一处理请求和响应，整个流程控制的中心，由它调用其它组件处理用户的请求

- HandlerMapping：**处理器映射器**，不需要工程师开发，由框架提供

作用：根据请求的url、method等信息查找Handler，即控制器方法

- Handler：**处理器**，需要工程师开发

作用：在DispatcherServlet的控制下Handler对具体的用户请求进行处理

- HandlerAdapter：**处理器适配器**，不需要工程师开发，由框架提供

作用：通过HandlerAdapter对处理器（控制器方法）进行执行

- ViewResolver：**视图解析器**，不需要工程师开发，由框架提供

作用：进行视图解析，得到相应的视图，例如：ThymeleafView、InternalResourceView、RedirectView

- View：**视图**

作用：将模型数据通过页面展示给用户

### DispatcherServlet初始化过程

DispatcherServlet 本质上是一个 Servlet，所以天然的遵循 Servlet 的生命周期。所以宏观上是 Servlet 生命周期来进行调度。

![image-20210826132821181](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210826132821181.png)

##### initWebApplicationContext

所在类：org.springframework.web.servlet.FrameworkServlet

```java
protected WebApplicationContext initWebApplicationContext() {
    WebApplicationContext rootContext =
        WebApplicationContextUtils.getWebApplicationContext(getServletContext());
    WebApplicationContext wac = null;

    if (this.webApplicationContext != null) {
        // A context instance was injected at construction time -> use it
        wac = this.webApplicationContext;
        if (wac instanceof ConfigurableWebApplicationContext) {
            ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
            if (!cwac.isActive()) {
                // The context has not yet been refreshed -> provide services such as
                // setting the parent context, setting the application context id, etc
                if (cwac.getParent() == null) {
                    // The context instance was injected without an explicit parent -> set
                    // the root application context (if any; may be null) as the parent
                    cwac.setParent(rootContext);
                }
                configureAndRefreshWebApplicationContext(cwac);
            }
        }
    }
    if (wac == null) {
        // No context instance was injected at construction time -> see if one
        // has been registered in the servlet context. If one exists, it is assumed
        // that the parent context (if any) has already been set and that the
        // user has performed any initialization such as setting the context id
        wac = findWebApplicationContext();
    }
    if (wac == null) {
        // No context instance is defined for this servlet -> create a local one
        // 创建WebApplicationContext
        wac = createWebApplicationContext(rootContext);
    }

    if (!this.refreshEventReceived) {
        // Either the context is not a ConfigurableApplicationContext with refresh
        // support or the context injected at construction time had already been
        // refreshed -> trigger initial onRefresh manually here.
        synchronized (this.onRefreshMonitor) {
            // 刷新WebApplicationContext
            onRefresh(wac);
        }
    }

    if (this.publishContext) {
        // Publish the context as a servlet context attribute.
        // 将IOC容器在应用域共享
        String attrName = getServletContextAttributeName();
        getServletContext().setAttribute(attrName, wac);
    }

    return wac;
}
```

##### WebApplicationContext

所在类：org.springframework.web.servlet.FrameworkServlet

```java
protected WebApplicationContext createWebApplicationContext(@Nullable ApplicationContext parent) {
    Class<?> contextClass = getContextClass();
    if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
        throw new ApplicationContextException(
            "Fatal initialization error in servlet with name '" + getServletName() +
            "': custom WebApplicationContext class [" + contextClass.getName() +
            "] is not of type ConfigurableWebApplicationContext");
    }
    // 通过反射创建 IOC 容器对象
    ConfigurableWebApplicationContext wac =
        (ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);

    wac.setEnvironment(getEnvironment());
    // 设置父容器
    wac.setParent(parent);
    String configLocation = getContextConfigLocation();
    if (configLocation != null) {
        wac.setConfigLocation(configLocation);
    }
    configureAndRefreshWebApplicationContext(wac);

    return wac;
}
```

##### DispatcherServlet初始化策略

FrameworkServlet创建WebApplicationContext后，刷新容器，调用onRefresh(wac)，此方法在DispatcherServlet中进行了重写，调用了initStrategies(context)方法，初始化策略，即初始化DispatcherServlet的各个组件

所在类：org.springframework.web.servlet.DispatcherServlet

```java
protected void initStrategies(ApplicationContext context) {
   initMultipartResolver(context);
   initLocaleResolver(context);
   initThemeResolver(context);
   initHandlerMappings(context);
   initHandlerAdapters(context);
   initHandlerExceptionResolvers(context);
   initRequestToViewNameTranslator(context);
   initViewResolvers(context);
   initFlashMapManager(context);
}
```





### DispatcherServlet调用组件处理请求

##### processRequest()

FrameworkServlet重写HttpServlet中的service()和doXxx()，这些方法中调用了processRequest(request, response)

所在类：org.springframework.web.servlet.FrameworkServlet

```java
protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
    throws ServletException, IOException {

    long startTime = System.currentTimeMillis();
    Throwable failureCause = null;

    LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
    LocaleContext localeContext = buildLocaleContext(request);

    RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
    ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
    asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());

    initContextHolders(request, localeContext, requestAttributes);

    try {
		// 执行服务，doService()是一个抽象方法，在DispatcherServlet中进行了重写
        doService(request, response);
    }
    catch (ServletException | IOException ex) {
        failureCause = ex;
        throw ex;
    }
    catch (Throwable ex) {
        failureCause = ex;
        throw new NestedServletException("Request processing failed", ex);
    }

    finally {
        resetContextHolders(request, previousLocaleContext, previousAttributes);
        if (requestAttributes != null) {
            requestAttributes.requestCompleted();
        }
        logResult(request, response, failureCause, asyncManager);
        publishRequestHandledEvent(request, response, startTime, failureCause);
    }
}
```

##### doService()

所在类：org.springframework.web.servlet.DispatcherServlet

```java
@Override
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
    logRequest(request);

    // Keep a snapshot of the request attributes in case of an include,
    // to be able to restore the original attributes after the include.
    Map<String, Object> attributesSnapshot = null;
    if (WebUtils.isIncludeRequest(request)) {
        attributesSnapshot = new HashMap<>();
        Enumeration<?> attrNames = request.getAttributeNames();
        while (attrNames.hasMoreElements()) {
            String attrName = (String) attrNames.nextElement();
            if (this.cleanupAfterInclude || attrName.startsWith(DEFAULT_STRATEGIES_PREFIX)) {
                attributesSnapshot.put(attrName, request.getAttribute(attrName));
            }
        }
    }

    // Make framework objects available to handlers and view objects.
    request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
    request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
    request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
    request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

    if (this.flashMapManager != null) {
        FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
        if (inputFlashMap != null) {
            request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
        }
        request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
        request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
    }

    RequestPath requestPath = null;
    if (this.parseRequestPath && !ServletRequestPathUtils.hasParsedRequestPath(request)) {
        requestPath = ServletRequestPathUtils.parseAndCache(request);
    }

    try {
        // 处理请求和响应
        doDispatch(request, response);
    }
    finally {
        if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
            // Restore the original attribute snapshot, in case of an include.
            if (attributesSnapshot != null) {
                restoreAttributesAfterInclude(request, attributesSnapshot);
            }
        }
        if (requestPath != null) {
            ServletRequestPathUtils.clearParsedRequestPath(request);
        }
    }
}
```

##### doDispatch()

所在类：org.springframework.web.servlet.DispatcherServlet

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

    try {
        ModelAndView mv = null;
        Exception dispatchException = null;

        try {
            processedRequest = checkMultipart(request);
            multipartRequestParsed = (processedRequest != request);

            // Determine handler for the current request.
            /*
            	mappedHandler：调用链
                包含handler、interceptorList、interceptorIndex
            	handler：浏览器发送的请求所匹配的控制器方法
            	interceptorList：处理控制器方法的所有拦截器集合
            	interceptorIndex：拦截器索引，控制拦截器afterCompletion()的执行
            */
            mappedHandler = getHandler(processedRequest);
            if (mappedHandler == null) {
                noHandlerFound(processedRequest, response);
                return;
            }

            // Determine handler adapter for the current request.
           	// 通过控制器方法创建相应的处理器适配器，调用所对应的控制器方法
            HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

            // Process last-modified header, if supported by the handler.
            String method = request.getMethod();
            boolean isGet = "GET".equals(method);
            if (isGet || "HEAD".equals(method)) {
                long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
                    return;
                }
            }
			
            // 调用拦截器的preHandle()
            if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                return;
            }

            // Actually invoke the handler.
            // 由处理器适配器调用具体的控制器方法，最终获得ModelAndView对象
            mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

            if (asyncManager.isConcurrentHandlingStarted()) {
                return;
            }

            applyDefaultViewName(processedRequest, mv);
            // 调用拦截器的postHandle()
            mappedHandler.applyPostHandle(processedRequest, response, mv);
        }
        catch (Exception ex) {
            dispatchException = ex;
        }
        catch (Throwable err) {
            // As of 4.3, we're processing Errors thrown from handler methods as well,
            // making them available for @ExceptionHandler methods and other scenarios.
            dispatchException = new NestedServletException("Handler dispatch failed", err);
        }
        // 后续处理：处理模型数据和渲染视图
        processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }
    catch (Exception ex) {
        triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
    }
    catch (Throwable err) {
        triggerAfterCompletion(processedRequest, response, mappedHandler,
                               new NestedServletException("Handler processing failed", err));
    }
    finally {
        if (asyncManager.isConcurrentHandlingStarted()) {
            // Instead of postHandle and afterCompletion
            if (mappedHandler != null) {
                mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
            }
        }
        else {
            // Clean up any resources used by a multipart request.
            if (multipartRequestParsed) {
                cleanupMultipart(processedRequest);
            }
        }
    }
}
```

##### processDispatchResult()

```java
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
                                   @Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
                                   @Nullable Exception exception) throws Exception {

    boolean errorView = false;

    if (exception != null) {
        if (exception instanceof ModelAndViewDefiningException) {
            logger.debug("ModelAndViewDefiningException encountered", exception);
            mv = ((ModelAndViewDefiningException) exception).getModelAndView();
        }
        else {
            Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
            mv = processHandlerException(request, response, handler, exception);
            errorView = (mv != null);
        }
    }

    // Did the handler return a view to render?
    if (mv != null && !mv.wasCleared()) {
        // 处理模型数据和渲染视图
        render(mv, request, response);
        if (errorView) {
            WebUtils.clearErrorRequestAttributes(request);
        }
    }
    else {
        if (logger.isTraceEnabled()) {
            logger.trace("No view rendering, null ModelAndView returned.");
        }
    }

    if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
        // Concurrent handling started during a forward
        return;
    }

    if (mappedHandler != null) {
        // Exception (if any) is already handled..
        // 调用拦截器的afterCompletion()
        mappedHandler.triggerAfterCompletion(request, response, null);
    }
}
```

### SpringMVC的执行流程

请求---> DispatcherServlet--->URL解析，找对应的映射

1. 不存在

   是否配置了mvc:default-servlet-handler

   没配 404

   配了找静态资源 找不到404

2. 存在

   HandlerMapping-->Handler以及Handler对象对应的拦截器, 以HandlerExecutionChain执行链对象返回

   Handler--->HandlerAdapter--->正向preHandler()

   提取Request中的模型数据，填充Handler入参，开始执行Handler（Controller)方法

   1. HttpMessageConveter： 将请求消息（如Json、xml等数据）转换成一个对象，将对象转换为指定的响应信息
   2. 数据转换：对请求消息进行数据转换。如String转换成Integer、Double等
   3. 数据格式化：对请求消息进行数据格式化。 如将字符串转换成格式化数字或格式化日期等
   4.  数据验证： 验证数据的有效性（长度、格式等），验证结果存储到BindingResult或Error中

   Handler执行完成后，向DispatcherServlet 返回一个ModelAndView对象。

   逆向postHandle()

   如果存在异常，则执行HandlerExceptionResolver进行异常处理）选择一个适合的ViewResolver进行视图解析，根据Model和View，来渲染视图。

   渲染视图完毕逆向执行afterCompletion()

   将渲染结果返回给客户端。

# SpringBoot2

https://yuque.com/atguigu/springboot 笔记

https://spring.io/projects/spring-boot#learn 文档



## 入门

https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started.html#getting-started.first-application  官方文档



## SpringBoot特点

### 依赖管理

父项目做依赖管理

```xml
依赖管理    
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.4.RELEASE</version>
</parent>

他的父项目
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.3.4.RELEASE</version>
</parent>

几乎声明了所有开发中常用的依赖的版本号,自动版本仲裁机制
```

引入依赖默认都可以不写版本 

引入非版本仲裁的jar，要写版本号

#### 自定义版本

1. 查看spring-boot-dependencies里面规定当前依赖的版本 用的 key。
2. 在当前项目里面重写配置

```xml
<properties>
    <java.version>1.8</java.version>
</properties>
```

#### starter

1. spring-boot-starter-* ： *就某种场景 

2. 只要引入starter，这个场景的所有常规需要的依赖我们都自动引入 

3. SpringBoot所有支持的场景 https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-starter 

4. 见到的  *-spring-boot-starter： 第三方为我们提供的简化开发的场景启动器。

5. 所有场景启动器最底层的依赖

   ```xml
   <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter</artifactId>
     <version>2.3.4.RELEASE</version>
     <scope>compile</scope>
   </dependency>
   ```

### 自动配置

- 自动配好Tomcat/SpringMVC/Web常见功能，如：字符编码问题

- ==默认扫描主程序所在包及其下面的所有子包里面的组件==

  ```java
  //自定义扫描路径
  @SpringBootApplication(scanBasePackages="com.atguigu")
  @ComponentScan 
  ```

- 各种配置拥有默认值
  - 默认配置最终都是映射到某个类上，如：MultipartProperties
  - ==配置文件的值最终会绑定每个类上，这个类会在容器中创建对象==

- ==按需加载所有自动配置项==

  - 非常多的starter
  - 引入了哪些场景这个场景的自动配置才会开启

  - SpringBoot所有的自动配置功能都在 ==spring-boot-autoconfigure 包里面==

#### 自动配置原理-加载

##### @SpringBootApplication

```java
@SpringBootConfiguration//代表当前是一个配置类
@EnableAutoConfiguration//关键: 开启自动配置
@ComponentScan(//包扫描
    excludeFilters = {@Filter(
        type = FilterType.CUSTOM,
        classes = {TypeExcludeFilter.class}
    ), @Filter(
        type = FilterType.CUSTOM,
        classes = {AutoConfigurationExcludeFilter.class}
    )}
)
public @interface SpringBootApplication {}
```

##### @EnableAutoConfiguration

```java
@AutoConfigurationPackage//自动配置包, 指定了默认的包规则
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {}
```

###### @AutoConfigurationPackage

```java
@Import({Registrar.class})
public @interface AutoConfigurationPackage {}
//利用Registrar给容器中导入一系列组件
//将指定的一个包下的所有组件导入进来---MainApplication 所在包下
public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
    AutoConfigurationPackages.register(registry, 
                                       (new AutoConfigurationPackages.PackageImport(metadata))
                                        .getPackageName());
}
```

###### @Import({AutoConfigurationImportSelector.class})

1. getAutoConfigurationEntry(annotationMetadata);给容器中批量导入一些组件 

2. List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes)获取到所有需要导入到容器中的配置类

3. 利用工厂加载 Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader)；得到所有的组件

4. 从META-INF/spring.factories位置来加载一个文件。 默认扫描我们当前系统里面所有META-INF/spring.factories位置的文件    

   spring-boot-autoconfigure-2.3.4.RELEASE.jar/META-INF/spring.factories 

   文件里面写死了spring-boot一启动就要给容器中加载的所有配置类  127个

#### 自动配置原理-按需开启

虽然我们127个场景的所有自动配置启动的时候默认全部加载。xxxxAutoConfiguration 

按照条件装配规则（@Conditional），最终会按需配置。

#### 修改默认配置

```java
@Bean
@ConditionalOnBean(MultipartResolver.class)  //容器中有这个类型组件
@ConditionalOnMissingBean(name = DispatcherServlet.MULTIPART_RESOLVER_BEAN_NAME) 
//容器中没有这个名字 multipartResolver 的组件
public MultipartResolver multipartResolver(MultipartResolver resolver) {
    //给@Bean标注的方法传入了对象参数，这个参数的值就会从容器中找。
    //SpringMVC multipartResolver。防止有些用户配置的文件上传解析器不符合规范
    // Detect if the user has created a MultipartResolver but named it incorrectly
    return resolver;
}
//给容器中加入了文件上传解析器；
```

==SpringBoot默认会在底层配好所有的组件。但是如果用户自己配置了以用户的优先==

```java
@Bean
@ConditionalOnMissingBean//关键
public CharacterEncodingFilter characterEncodingFilter() {}
```

总结：

- SpringBoot先加载所有的自动配置类  xxxxxAutoConfiguration
- 每个自动配置类按照条件进行生效，默认都会绑定配置文件指定的值。xxxxProperties里面拿。xxxProperties和配置文件进行了绑定

- 生效的配置类就会给容器中装配很多组件
- 只要容器中有这些组件，相当于这些功能就有了

- 定制化配置
  - 用户直接自己@Bean替换底层的组件
  - 用户去看这个组件是获取的配置文件什么值就去修改。



**xxxxxAutoConfiguration ---> 组件  --->** **xxxxProperties里面拿值  ----> application.properties**



#### 最佳实践

- 引入场景依赖
  - https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-starter

- 查看自动配置了哪些（选做）
  - 自己分析，引入场景对应的自动配置一般都生效了
  - 配置文件中debug=true开启自动配置报告。Negative（不生效）\Positive（生效）

- 是否需要修改
  - 参照文档修改配置项
    - https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html#common-application-properties
    - 自己分析。xxxxProperties绑定了配置文件的哪些。
  - 自定义加入或者替换组件
    - @Bean、@Component。。。
  - 自定义器  **XXXXXCustomizer**；
  - ......

## IOC

### bean添加

```java
@Bean
@Component
@Controller
@Service
@Repository
@ComponentScan
```

#### @Configuration

**Full模式与Lite模式**

- 配置类bean之间无依赖关系用Lite模式加速容器启动过程，减少判断
- 配置类bean之间有依赖关系，方法会被调用得到之前单实例组件，用Full模式

```java
#############################Configuration使用示例########
/**
 * 1、@Bean给容器注册组件，默认单实例
 * 2、配置类本身也是组件
 * 3、proxyBeanMethods：代理bean的方法
 *      Full(proxyBeanMethods = true)、【保证每个@Bean方法被调用多少次返回的组件都是单实例的】
 *      Lite(proxyBeanMethods = false)【每个@Bean方法被调用多少次返回的组件都是新创建的】
 *      组件依赖必须使用Full模式默认。其他默认是否Lite模式
 */
@Configuration(proxyBeanMethods = false) //告诉SpringBoot这是一个配置类 == 配置文件
public class MyConfig {
    /**
     * Full:外部无论对配置类中的这个组件注册方法调用多少次获取的都是之前注册容器中的单实例对象
     */
    @Bean //给容器中添加组件。以方法名作为组件的id。返回类型就是组件类型。返回的值，就是组件在容器中的实例
    public User user01(){
        User zhangsan = new User("zhangsan", 18);
        //user组件依赖了Pet组件
        zhangsan.setPet(tomcatPet());
        return zhangsan;
    }
    @Bean("tom")
    public Pet tomcatPet(){
        return new Pet("tomcat");
    }
}
################################@Configuration测试代码如下########################################
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan("com.atguigu.boot")
public class MainApplication {
    public static void main(String[] args) {
        //1、返回我们IOC容器
        ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class, args);
        //2、查看容器里面的组件
        String[] names = run.getBeanDefinitionNames();
        for (String name : names) {
            System.out.println(name);
        }
        //3、从容器中获取组件
        Pet tom01 = run.getBean("tom", Pet.class);
        Pet tom02 = run.getBean("tom", Pet.class);
        System.out.println("组件："+(tom01 == tom02));
        //4、com.atguigu.boot.config.MyConfig$$EnhancerBySpringCGLIB$$51f1e1ca@1654a892
        MyConfig bean = run.getBean(MyConfig.class);//Full模式,获取的是代理对象
        System.out.println(bean);
        //如果@Configuration(proxyBeanMethods = true)代理对象调用方法。SpringBoot总会检查这个组件是否在容器中有。
        //保持组件单实例
        User user = bean.user01();
        User user1 = bean.user01();
        System.out.println(user == user1);

        User user01 = run.getBean("user01", User.class);
        Pet tom = run.getBean("tom", Pet.class);
        System.out.println("用户的宠物："+(user01.getPet() == tom));
    }
}
```

#### @Import

给容器中自动创建出这两个类型的bean,   默认id是全类名   

```java
@Import({User.class, DBHelper.class})
```

#### @==Conditional==

条件装配：满足Conditional指定的条件，则进行bean注入

![image-20210820142537219](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210820142537219.png)

```java
=====================测试条件装配==========================
    @Configuration(proxyBeanMethods = false)
    //@ConditionalOnBean(name = "tom")
    @ConditionalOnMissingBean(name = "tom")//方法上管一个,类上管类中所有bean
    public class MyConfig {
        @Bean
        public User user01(){
            User zhangsan = new User("zhangsan", 18);
            //user组件依赖了Pet组件
            zhangsan.setPet(tomcatPet());
            return zhangsan;
        }
        @Bean("tom")
        public Pet tomcatPet(){
            return new Pet("tomcat");
        }
    }

public static void main(String[] args) {
    //1、返回我们IOC容器
    ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class, args);
    //2、查看容器里面的组件
    String[] names = run.getBeanDefinitionNames();
    for (String name : names) {
        System.out.println(name);
    }
    boolean tom = run.containsBean("tom");
    System.out.println("容器中Tom组件："+tom);
    boolean user01 = run.containsBean("user01");
    System.out.println("容器中user01组件："+user01);
    boolean tom22 = run.containsBean("tom22");
    System.out.println("容器中tom22组件："+tom22);
}
```

#### @ImportResource

导入xml配置的bean

```java
@ImportResource("classpath:beans.xml")
```

#### 配置绑定

properties文件中的配置，封装到JavaBean

##### @ConfigurationProperties&@Component

```java
@Component//只有在容器中的组件，才会拥有SpringBoot提供的强大功能
@ConfigurationProperties(prefix = "mycar")
public class Car {

    private String brand;
    private Integer price;
}
```

```properties
mycar.brand=123
mycar.price=1234
```

##### @EnableConfigurationProperties&@ConfigurationProperties

```java
@EnableConfigurationProperties(Car.class)
//1、开启Car配置绑定功能
//2、把这个Car这个组件自动注册到容器中
public class MyConfig {
}
```

## 开发小技巧

### Lombok

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
idea中搜索安装lombok插件
```

```java
===============================简化JavaBean开发===================================
@NoArgsConstructor
//@AllArgsConstructor
@Data
@ToString
@EqualsAndHashCode
public class User {
    private String name;
    private Integer age;
    private Pet pet;
    public User(String name,Integer age){
        this.name = name;
        this.age = age;
    }
}
================================简化日志开发===================================
@Slf4j
@RestController
public class HelloController {
    @RequestMapping("/hello")
    public String handle01(@RequestParam("name") String name){
        log.info("请求进来了....");       
        return "Hello, Spring Boot 2!"+"你好："+name;
    }
}
```

### dev-tools/JRebel

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

项目或者页面修改以后：Ctrl+F9

### Spring Initailizr（项目初始化向导）

自动依赖引入

自动创建项目结构

自动编写好主配置类

## 核心功能

### 配置文件

#### properties

#### yaml

一种标记语言

##### 基本语法

- key: value；kv之间有空格
- 大小写敏感

- 使用缩进表示层级关系
- 缩进不允许使用tab，只允许空格

- 缩进的空格数不重要，只要相同层级的元素左对齐即可
- '#'表示注释

- 字符串无需加引号，如果要加，''与""表示字符串内容 会被 转义/不转义

##### 数据类型

- 字面量：单个的、不可再分的值。date、boolean、string、number、null

  ```yml
  k: v
  ```

* 对象：键值对的集合。map、hash、set、object 

  ```yaml
  行内写法：  k: {k1:v1,k2:v2,k3:v3}
  #或
  k: 
    k1: v1
    k2: v2
    k3: v3
  ```

- 数组：一组按次序排列的值。array、list、queue

  ```yaml
  行内写法：  k: [v1,v2,v3]
  #或者
  k:
   - v1
   - v2
   - v3
  ```

##### 示例

```java
@Data
public class Person {
	private String userName;
	private Boolean boss;
	private Date birth;
	private Integer age;
	private Pet pet;
	private String[] interests;
	private List<String> animal;
	private Map<String, Object> score;
	private Set<Double> salarys;
	private Map<String, List<Pet>> allPets;
}
@Data
public class Pet {
	private String name;
	private Double weight;
}
```

```yaml
# yaml表示以上对象
person:
  userName: zhangsan
  boss: false
  birth: 2019/12/12 20:12:33
  age: 18
  pet: 
    name: tomcat
    weight: 23.4
  interests: [篮球,游泳]
  animal: 
    - jerry
    - mario
  score:
    english: 
      first: 30
      second: 40
      third: 50
    math: [131,140,148]
    chinese: {first: 128,second: 136}
  salarys: [3999,4999.98,5999.99]
  allPets:
    sick:
      - {name: tom}
      - {name: jerry,weight: 47}
    health: [{name: mario,weight: 47}]
```

#### 配置提示

自定义的类和配置文件绑定一般没有提示。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
<!--打包时去掉-->
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <excludes>
                    <exclude>
                        <groupId>org.springframework.boot</groupId>
                        <artifactId>spring-boot-configuration-processor</artifactId>
                    </exclude>
                </excludes>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### web开发

#### 静态资源

##### 访问

只要静态资源放在类路径下： 

```
/static
/public
/resources
/META-INF/resources
```

访问 ： 当前项目根路径/ + 静态资源名 



原理： 静态映射/**

请求进来，先去找Controller看能不能处理。

不能处理的所有请求又都交给静态资源处理器。静态资源也找不到则响应404页面

```yaml

spring:
  mvc:
    static-path-pattern: /res/** #改变静态资源访问前缀

  resources:
    static-locations: [classpath:/haha/] #改变默认的静态资源路径
```

###### webjar

自动映射 /[webjars](http://localhost:8080/webjars/jquery/3.5.1/jquery.js)/**
https://www.webjars.org/

```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.5.1</version>
</dependency>
```

访问地址：[http://localhost:8080/webjars/**jquery/3.5.1/jquery.js**](http://localhost:8080/webjars/jquery/3.5.1/jquery.js)   后面地址要按照依赖里面的包路径

###### 欢迎页支持

- 静态资源路径下  index.html
  - 可以配置静态资源路径
  - 但是不可以配置静态资源的访问前缀。否则导致 index.html不能被默认访问

```yaml
spring:
#  mvc:
#    static-path-pattern: /res/**   这个会导致welcome page功能失效
  resources:
    static-locations: [classpath:/haha/]
```

- controller能处理/index

###### 自定义 Favicon

favicon.ico 放在静态资源目录下即可。

```yaml
spring:
#  mvc:
#    static-path-pattern: /res/**   这个会导致 Favicon 功能失效
```

##### 静态资源配置原理

- SpringBoot启动默认加载  xxxAutoConfiguration 类（自动配置类）
- SpringMVC功能的自动配置类 WebMvcAutoConfiguration，生效

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {}
```

- 给容器中配了什么。

```java
@Configuration(proxyBeanMethods = false)
@Import(EnableWebMvcConfiguration.class)
@EnableConfigurationProperties({ WebMvcProperties.class, ResourceProperties.class })
@Order(0)
public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer {}
```

- 配置文件的相关属性和xxx进行了绑定。

  WebMvcProperties==**spring.mvc**、

  ResourceProperties==**spring.resources**



###### 配置类只有一个有参构造器

```java
//有参构造器所有参数的值都会从容器中确定
//ResourceProperties resourceProperties；获取和spring.resources绑定的所有的值的对象
//WebMvcProperties mvcProperties 获取和spring.mvc绑定的所有的值的对象
//ListableBeanFactory beanFactory Spring的beanFactory
//HttpMessageConverters 找到所有的HttpMessageConverters
//ResourceHandlerRegistrationCustomizer 找到 资源处理器的自定义器。=========
//DispatcherServletPath  
//ServletRegistrationBean   给应用注册Servlet、Filter....
public WebMvcAutoConfigurationAdapter(
    	ResourceProperties resourceProperties, 
		WebMvcProperties mvcProperties,
		ListableBeanFactory beanFactory, 
    	ObjectProvider<HttpMessageConverters> messageConvertersProvider,
    	ObjectProvider<ResourceHandlerRegistrationCustomizer> 			
    					resourceHandlerRegistrationCustomizerProvider,
    	ObjectProvider<DispatcherServletPath> dispatcherServletPath,
    	ObjectProvider<ServletRegistrationBean<?>> servletRegistrations) {
    this.resourceProperties = resourceProperties;
    this.mvcProperties = mvcProperties;
    this.beanFactory = beanFactory;
    this.messageConvertersProvider = messageConvertersProvider;
    this.resourceHandlerRegistrationCustomizer = resourceHandlerRegistrationCustomizerProvider.getIfAvailable();
    this.dispatcherServletPath = dispatcherServletPath;
    this.servletRegistrations = servletRegistrations;
}
```

###### 资源处理的默认规则

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    if (!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
        return;
    }
    Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
    CacheControl cacheControl = 
        this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
    //webjars的规则
    if (!registry.hasMappingForPattern("/webjars/**")) {
        customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
             	.addResourceLocations("classpath:/META-INF/resources/webjars/")                                 .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
    }
    //
    String staticPathPattern = this.mvcProperties.getStaticPathPattern();
    if (!registry.hasMappingForPattern(staticPathPattern)) {
        customizeResourceHandlerRegistration(registry
                .addResourceHandler(staticPathPattern)
                .addResourceLocations(getResourceLocations(
                        this.resourceProperties.getStaticLocations()))
                .setCachePeriod(getSeconds(cachePeriod))
                .setCacheControl(cacheControl));
    }
}
```

```yaml
spring:
#  mvc:
#    static-path-pattern: /res/**
  resources:
    add-mappings: false   禁用所有静态资源规则
```

```java
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {

	private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { 
        "classpath:/META-INF/resources/","classpath:/resources/",
        "classpath:/static/", "classpath:/public/" };
	/**
	 *静态资源默认的四个位置
	 */
	private String[] staticLocations = CLASSPATH_RESOURCE_LOCATIONS;
```

###### 欢迎页的处理规则

```java
HandlerMapping：处理器映射。保存了每一个Handler能处理哪些请求。	

@Bean
public WelcomePageHandlerMapping welcomePageHandlerMapping(
        ApplicationContext applicationContext,
        FormattingConversionService mvcConversionService, 
        ResourceUrlProvider mvcResourceUrlProvider) {
    WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(
        new TemplateAvailabilityProviders(applicationContext), 
        applicationContext, getWelcomePage(),this.mvcProperties.getStaticPathPattern());
    welcomePageHandlerMapping.setInterceptors(getInterceptors(
        mvcConversionService, mvcResourceUrlProvider));
    welcomePageHandlerMapping.setCorsConfigurations(getCorsConfigurations());
    return welcomePageHandlerMapping;
}

WelcomePageHandlerMapping(TemplateAvailabilityProviders templateAvailabilityProviders,
                          ApplicationContext applicationContext, 
                          Optional<Resource> welcomePage, 
                          String staticPathPattern) {
    if (welcomePage.isPresent() && "/**".equals(staticPathPattern)) {
        //要用欢迎页功能，必须是/**
        logger.info("Adding welcome page: " + welcomePage.get());
        setRootViewName("forward:index.html");
    }
    else if (welcomeTemplateExists(templateAvailabilityProviders, applicationContext)) {
        // 调用Controller  /index
        logger.info("Adding welcome page template: index");
        setRootViewName("index");
    }
}
```

#### Request参数处理

##### rest

- GET-获取用户   DELETE-删除用户     PUT-修改用户    POST-保存用户

- 核心Filter；HiddenHttpMethodFilter
  - 用法： 表单method=post，隐藏域 <input name="_method" value="PUT">

  - SpringBoot中手动开启

    ```yaml
    spring:
      mvc:
        hiddenmethod:
          filter:
            enabled: true   #开启页面表单的Rest功能
    ```

- 扩展：如何把_method 这个名字换成我们自己喜欢的。

###### Rest原理（表单提交）

- 表单提交会带上**_method=PUT**
- **请求过来被**HiddenHttpMethodFilter拦截
  - 请求是否正常，并且是POST
  - 获取到**_method**的值。
  - 兼容以下请求；**PUT**.**DELETE**.**PATCH**
  - **原生request（post），包装模式requesWrapper重写了getMethod方法，返回的是传入的值。**
  - **过滤器链放行的时候用wrapper。以后的方法调用getMethod是调用requesWrapper的。**

```java
@Bean//源码
@ConditionalOnMissingBean(HiddenHttpMethodFilter.class)
@ConditionalOnProperty(prefix = "spring.mvc.hiddenmethod.filter", name = "enabled", matchIfMissing = false)
public OrderedHiddenHttpMethodFilter hiddenHttpMethodFilter() {
    return new OrderedHiddenHttpMethodFilter();
}
//自定义filter
@Bean
public HiddenHttpMethodFilter hiddenHttpMethodFilter(){
    HiddenHttpMethodFilter methodFilter = new HiddenHttpMethodFilter();
    methodFilter.setMethodParam("_m");
    return methodFilter;
}
```

**Rest使用客户端工具**

- 如PostMan直接发送Put、delete等方式请求，无需Filter。

##### 请求映射原理

doGet/doPost -> processRequest -> doService -> doDispatch

![image-20210820174140458](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210820174140458.png)

SpringMVC功能分析都从 org.springframework.web.servlet.DispatcherServlet   --->  doDispatch（）

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

    try {
        ModelAndView mv = null;
        Exception dispatchException = null;

        try {
            processedRequest = checkMultipart(request);
            multipartRequestParsed = (processedRequest != request);

            // 找到当前请求使用哪个Handler（Controller的方法）处理
            mappedHandler = getHandler(processedRequest);

            //HandlerMapping：处理器映射。/xxx->>xxxx
            ...
        }
    }
    @Nullable
    protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
        if (this.handlerMappings != null) {
            Iterator var2 = this.handlerMappings.iterator();

            while(var2.hasNext()) {
                HandlerMapping mapping = (HandlerMapping)var2.next();
                HandlerExecutionChain handler = mapping.getHandler(request);
                if (handler != null) {
                    return handler;
                }
            }
        }

        return null;
    }
```

![image-20210820174556035](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210820174556035.png)

**RequestMappingHandlerMapping**：保存了所有@RequestMapping 和handler的映射规则。

![image-20210820174837977](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210820174837977.png)

所有的请求映射都在HandlerMapping中。

- SpringBoot自动配置欢迎页的 WelcomePageHandlerMapping 。访问 /能访问到index.html；
- SpringBoot自动配置了默认 的 RequestMappingHandlerMapping

- 请求进来，挨个尝试所有的HandlerMapping看是否有请求信息。
  - 如果有就找到这个请求对应的handler
  - 如果没有就是下一个 HandlerMapping

- 我们需要一些自定义的映射处理，我们也可以自己给容器中放**HandlerMapping**。自定义 **HandlerMapping**

##### 普通参数与基本注解

###### 1.注解

```java
@PathVariable //Map<String,String> pv 可以传入到map
@RequestHeader //获取请求头 map获取所有
@ModelAttribute、
@RequestParam //map取所有
@MatrixVariable//矩阵变量
@CookieValue //Cookie 取所有
@RequestBody//获取请求体[POST]

@RequestAttribute//获取请求域中属性值
```

```java
@RestController
public class ParameterTestController {
    //  car/2/owner/zhangsan
    @GetMapping("/car/{id}/owner/{username}")
    public Map<String,Object> getCar(@PathVariable("id") Integer id,
                                     @PathVariable("username") String name,
                                     @PathVariable Map<String,String> pv,
                                     @RequestHeader("User-Agent") String userAgent,
                                     @RequestHeader Map<String,String> header,
                                     @RequestParam("age") Integer age,
                                     @RequestParam("inters") List<String> inters,
                                     @RequestParam Map<String,String> params,
                                     @CookieValue("_ga") String _ga,
                                     @CookieValue("_ga") Cookie cookie){
        Map<String,Object> map = new HashMap<>();
        //        map.put("id",id);
        //        map.put("name",name);
        //        map.put("pv",pv);
        //        map.put("userAgent",userAgent);
        //        map.put("headers",header);
        map.put("age",age);
        map.put("inters",inters);
        map.put("params",params);
        map.put("_ga",_ga);
        System.out.println(cookie.getName()+"===>"+cookie.getValue());
        return map;
    }
    @PostMapping("/save")
    public Map postMethod(@RequestBody String content){
        Map<String,Object> map = new HashMap<>();
        map.put("content",content);
        return map;
    }
    //1、语法： 请求路径：/cars/sell;low=34;brand=byd,audi,yd
    //2、SpringBoot默认是禁用了矩阵变量的功能
    //      手动开启：原理。对于路径的处理。UrlPathHelper进行解析。
    //              removeSemicolonContent（移除分号内容）支持矩阵变量的
    //3、矩阵变量必须有url路径变量才能被解析
    //页面开发，cookie禁用了，session里面的内容怎么使用；
    //session.set(a,b)---> jsessionid ---> cookie ----> 每次发请求携带。
    //url重写：/abc;jsesssionid=xxxx 把cookie的值使用矩阵变量的方式进行传递.
    //
    @GetMapping("/cars/{path}")
    public Map carsSell(@MatrixVariable("low") Integer low,
                        @MatrixVariable("brand") List<String> brand,
                        @PathVariable("path") String path){
        Map<String,Object> map = new HashMap<>();

        map.put("low",low);
        map.put("brand",brand);
        map.put("path",path);
        return map;
    }

    // /boss/1;age=20/2;age=10
    @GetMapping("/boss/{bossId}/{empId}")
    public Map boss(@MatrixVariable(value = "age",pathVar = "bossId") Integer bossAge,
                    @MatrixVariable(value = "age",pathVar = "empId") Integer empAge){
        Map<String,Object> map = new HashMap<>();

        map.put("bossAge",bossAge);
        map.put("empAge",empAge);
        return map;
    }
}
@Bean
public WebMvcConfigurer webMvcConfigurer(){
    return new WebMvcConfigurer() {
        @Override//开启矩阵变量
        public void configurePathMatch(PathMatchConfigurer configurer) {
            UrlPathHelper urlPathHelper = new UrlPathHelper();
            // 不移除；后面的内容。矩阵变量功能就可以生效
            urlPathHelper.setRemoveSemicolonContent(false);
            configurer.setUrlPathHelper(urlPathHelper);
        }
    }
}
```

```java
@Controller
public class RequestController {
    @GetMapping("/goto")
    public String goToPage(HttpServletRequest request){
        request.setAttribute("msg","成功了...");
        request.setAttribute("code",200);
        return "forward:/success";  //转发到  /success请求
    }
    @GetMapping("/params")
    public String testParam(Map<String,Object> map,
                            Model model,
                            HttpServletRequest request,
                            HttpServletResponse response){
        map.put("hello","world666");
        model.addAttribute("world","hello666");
        request.setAttribute("message","HelloWorld");
        Cookie cookie = new Cookie("c1","v1");
        response.addCookie(cookie);
        return "forward:/success";
    }
    @ResponseBody
    @GetMapping("/success")
    public Map success(@RequestAttribute(value = "msg",required = false) String msg,
                       @RequestAttribute(value = "code",required = false)Integer code,
                       HttpServletRequest request){
        Object msg1 = request.getAttribute("msg");

        Map<String,Object> map = new HashMap<>();
        Object hello = request.getAttribute("hello");
        Object world = request.getAttribute("world");
        Object message = request.getAttribute("message");

        map.put("reqMethod_msg",msg1);
        map.put("annotation_msg",msg);
        map.put("hello",hello);
        map.put("world",world);
        map.put("message",message);

        return map;
    }
}
```

###### 2.Servlet API：

```java
WebRequest
ServletRequest
MultipartRequest
HttpSession
javax.servlet.http.PushBuilder
Principal
InputStream
Reader
HttpMethod
Locale
TimeZone
ZoneId
```



**ServletRequestMethodArgumentResolver  以上的部分参数**

```java
@Override
	public boolean supportsParameter(MethodParameter parameter) {
		Class<?> paramType = parameter.getParameterType();
		return (WebRequest.class.isAssignableFrom(paramType) ||
				ServletRequest.class.isAssignableFrom(paramType) ||
				MultipartRequest.class.isAssignableFrom(paramType) ||
				HttpSession.class.isAssignableFrom(paramType) ||
				(pushBuilder != null && pushBuilder.isAssignableFrom(paramType)) ||
				Principal.class.isAssignableFrom(paramType) ||
				InputStream.class.isAssignableFrom(paramType) ||
				Reader.class.isAssignableFrom(paramType) ||
				HttpMethod.class == paramType ||
				Locale.class == paramType ||
				TimeZone.class == paramType ||
				ZoneId.class == paramType);
	}
```

###### 3.复杂参数

```java
Map/Model/HttpServletRequest//都是可以给request域中放数据  request.setAttribute
Errors/BindingResult
RedirectAttributes//重定向携带数据
ServletResponse//response
SessionStatus
UriComponentsBuilder
ServletUriComponentsBuilder
```

**Map、Model类型的参数**，会返回 mavContainer.getModel（）；---> BindingAwareModelMap 是Model 也是Map

**mavContainer**.getModel(); 获取到值的

###### 4.自定义对象参数

可以自动类型转换与格式化，可以级联封装

```java
/**
 *     姓名： <input name="userName"/> <br/>
 *     年龄： <input name="age"/> <br/>
 *     生日： <input name="birth"/> <br/>
 *     宠物姓名：<input name="pet.name"/><br/>
 *     宠物年龄：<input name="pet.age"/>
 */
@Data
public class Person {
    private String userName;
    private Integer age;
    private Date birth;
    private Pet pet;   
}
@Data
public class Pet {
    private String name;
    private String age;
}

result
    /**
     * 数据绑定：页面提交的请求数据（GET、POST）都可以和对象属性进行绑定
     */
    @PostMapping("/saveuser")
    public Person saveuser(Person person){

    return person;
}
```

##### 参数处理原理

- HandlerMapping中找到能处理请求的Handler（Controller.method()）
- 为当前Handler 找一个适配器 HandlerAdapter； **RequestMappingHandlerAdapter**

- 适配器执行目标方法并确定方法参数的每一个值



###### 1. HandlerAdapter

```java
RequestMappingHandlerAdapter//支持方法上标注@RequestMapping
HandlerFunctionAdapter//函数式编程的
HttpRequestHandlerAdapter
SimpleControllerHandlerAdapter
```

###### 2. 执行目标方法

```java
// Actually invoke the handler.
//DispatcherServlet -- doDispatch
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
```

```java

mav = invokeHandlerMethod(request, response, handlerMethod); //执行目标方法


//ServletInvocableHandlerMethod真正执行目标方法
Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);
//获取方法的参数值
Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);
```

###### 3. 参数解析器-HandlerMethodArgumentResolver-26个

确定将要执行的目标方法的每一个参数的值是什么;

SpringMVC目标方法能写多少种参数类型。取决于参数解析器, 

![image-20210823135009742](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210823135009742.png)

```java
resolveArgument()//支持就调用 resolveArgument
supportsParameter()//当前解析器是否支持解析这种参数
```

###### 4. 返回值处理器-15个

![image-20210823135822418](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210823135822418.png)

###### 5. 如何确定目标方法每一个参数的值

```java
============InvocableHandlerMethod==========================
protected Object[] getMethodArgumentValues(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
			Object... providedArgs) throws Exception {

		MethodParameter[] parameters = getMethodParameters();
		if (ObjectUtils.isEmpty(parameters)) {
			return EMPTY_ARGS;
		}

		Object[] args = new Object[parameters.length];
		for (int i = 0; i < parameters.length; i++) {
			MethodParameter parameter = parameters[i];
			parameter.initParameterNameDiscovery(this.parameterNameDiscoverer);
			args[i] = findProvidedArgument(parameter, providedArgs);
			if (args[i] != null) {
				continue;
			}
			if (!this.resolvers.supportsParameter(parameter)) {
				throw new IllegalStateException(formatArgumentError(parameter, "No suitable resolver"));
			}
			try {
				args[i] = this.resolvers.resolveArgument(parameter, mavContainer, request, this.dataBinderFactory);
			}
			catch (Exception ex) {
				// Leave stack trace for later, exception may actually be resolved and handled...
				if (logger.isDebugEnabled()) {
					String exMsg = ex.getMessage();
					if (exMsg != null && !exMsg.contains(parameter.getExecutable().toGenericString())) {
						logger.debug(formatArgumentError(parameter, exMsg));
					}
				}
				throw ex;
			}
		}
		return args;
	}
```

5.1 挨个判断所有参数解析器那个支持解析这个参数

```java
@Nullable
private HandlerMethodArgumentResolver getArgumentResolver(MethodParameter parameter) {
    HandlerMethodArgumentResolver result = this.argumentResolverCache.get(parameter);
    if (result == null) {
        for (HandlerMethodArgumentResolver resolver : this.argumentResolvers) {
            if (resolver.supportsParameter(parameter)) {
                result = resolver;
                this.argumentResolverCache.put(parameter, result);
                break;
            }
        }
    }
    return result;
}
```

5.2 解析这个参数的值

```java
调用各自 HandlerMethodArgumentResolver 的 resolveArgument 方法即可
```

5.3 自定义类型参数 封装POJO

**ServletModelAttributeMethodProcessor  这个参数处理器支持**

 **是否为简单类型。**

```java
public static boolean isSimpleValueType(Class<?> type) {
		return (Void.class != type && void.class != type &&
				(ClassUtils.isPrimitiveOrWrapper(type) ||
				Enum.class.isAssignableFrom(type) ||
				CharSequence.class.isAssignableFrom(type) ||
				Number.class.isAssignableFrom(type) ||
				Date.class.isAssignableFrom(type) ||
				Temporal.class.isAssignableFrom(type) ||
				URI.class == type ||
				URL.class == type ||
				Locale.class == type ||
				Class.class == type));
	}
```

```java
@Override
	@Nullable
	public final Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
			NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {

		Assert.state(mavContainer != null, "ModelAttributeMethodProcessor requires ModelAndViewContainer");
		Assert.state(binderFactory != null, "ModelAttributeMethodProcessor requires WebDataBinderFactory");

		String name = ModelFactory.getNameForParameter(parameter);
		ModelAttribute ann = parameter.getParameterAnnotation(ModelAttribute.class);
		if (ann != null) {
			mavContainer.setBinding(name, ann.binding());
		}

		Object attribute = null;
		BindingResult bindingResult = null;

		if (mavContainer.containsAttribute(name)) {
			attribute = mavContainer.getModel().get(name);
		}
		else {
			// Create attribute instance
			try {
				attribute = createAttribute(name, parameter, binderFactory, webRequest);
			}
			catch (BindException ex) {
				if (isBindExceptionRequired(parameter)) {
					// No BindingResult parameter -> fail with BindException
					throw ex;
				}
				// Otherwise, expose null/empty value and associated BindingResult
				if (parameter.getParameterType() == Optional.class) {
					attribute = Optional.empty();
				}
				bindingResult = ex.getBindingResult();
			}
		}

		if (bindingResult == null) {
			// Bean property binding and validation;
			// skipped in case of binding failure on construction.
			WebDataBinder binder = binderFactory.createBinder(webRequest, attribute, name);
			if (binder.getTarget() != null) {
				if (!mavContainer.isBindingDisabled(name)) {
					bindRequestParameters(binder, webRequest);
				}
				validateIfApplicable(binder, parameter);
				if (binder.getBindingResult().hasErrors() && isBindExceptionRequired(binder, parameter)) {
					throw new BindException(binder.getBindingResult());
				}
			}
			// Value type adaptation, also covering java.util.Optional
			if (!parameter.getParameterType().isInstance(attribute)) {
				attribute = binder.convertIfNecessary(binder.getTarget(), parameter.getParameterType(), parameter);
			}
			bindingResult = binder.getBindingResult();
		}

		// Add resolved attribute and BindingResult at the end of the model
		Map<String, Object> bindingResultModel = bindingResult.getModel();
		mavContainer.removeAttributes(bindingResultModel);
		mavContainer.addAllAttributes(bindingResultModel);

		return attribute;
	}
```

**WebDataBinder binder = binderFactory.createBinder(webRequest, attribute, name);**

**WebDataBinder :web数据绑定器，将请求参数的值绑定到指定的JavaBean里面**

**WebDataBinder 利用它里面的 Converters 将请求数据转成指定的数据类型。再次封装到JavaBean中**



**GenericConversionService：在设置每一个值的时候，找它里面的所有converter那个可以将这个数据类型（request带来参数的字符串）转换到指定的类型（JavaBean -- Integer）**

**byte -- > file**



@FunctionalInterface**public interface** Converter<S, T>

未来我们可以给WebDataBinder里面放自己的Converter；

**private static final class** StringToNumber<T **extends** Number> **implements** Converter<String, T>



自定义 Converter

```java
    //1、WebMvcConfigurer定制化SpringMVC的功能
    @Bean
    public WebMvcConfigurer webMvcConfigurer(){
        return new WebMvcConfigurer() {
            @Override
            public void configurePathMatch(PathMatchConfigurer configurer) {
                UrlPathHelper urlPathHelper = new UrlPathHelper();
                // 不移除；后面的内容。矩阵变量功能就可以生效
                urlPathHelper.setRemoveSemicolonContent(false);
                configurer.setUrlPathHelper(urlPathHelper);
            }

            @Override
            public void addFormatters(FormatterRegistry registry) {
                registry.addConverter(new Converter<String, Pet>() {

                    @Override
                    public Pet convert(String source) {
                        // 啊猫,3
                        if(!StringUtils.isEmpty(source)){
                            Pet pet = new Pet();
                            String[] split = source.split(",");
                            pet.setName(split[0]);
                            pet.setAge(Integer.parseInt(split[1]));
                            return pet;
                        }
                        return null;
                    }
                });
            }
        };
    }
```







###### 6. 目标方法执行完成

将所有的数据都放在 **ModelAndViewContainer**；包含要去的页面地址View。还包含Model数据。

###### 7. 处理派发结果

```java
processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);

renderMergedOutputModel(mergedModel, getRequestToExpose(request), response);
```

```java
//InternalResourceView：
    @Override
    protected void renderMergedOutputModel(
    Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
	//暴露模型作为请求域属性
    // Expose the model object as request attributes.
    exposeModelAsRequestAttributes(model, request);

    // Expose helpers as request attributes, if any.
    exposeHelpers(request);

    // Determine the path for the request dispatcher.
    String dispatcherPath = prepareForRendering(request, response);

    // Obtain a RequestDispatcher for the target resource (typically a JSP).
    RequestDispatcher rd = getRequestDispatcher(request, dispatcherPath);
    if (rd == null) {
        throw new ServletException("Could not get RequestDispatcher for [" + getUrl() +
                                   "]: Check that the corresponding file exists within your web application archive!");
    }

    // If already included or response already committed, perform include, else forward.
    if (useInclude(request, response)) {
        response.setContentType(getContentType());
        if (logger.isDebugEnabled()) {
            logger.debug("Including [" + getUrl() + "]");
        }
        rd.include(request, response);
    }

    else {
        // Note: The forwarded resource is supposed to determine the content type itself.
        if (logger.isDebugEnabled()) {
            logger.debug("Forwarding to [" + getUrl() + "]");
        }
        rd.forward(request, response);
    }
}
```

```java
protected void exposeModelAsRequestAttributes(Map<String, Object> model,
			HttpServletRequest request) throws Exception {

    //model中的所有数据遍历挨个放在请求域中
		model.forEach((name, value) -> {
			if (value != null) {
				request.setAttribute(name, value);
			}
			else {
				request.removeAttribute(name);
			}
		});
	}
```

#### 数据响应与内容协商

##### 响应JSON

###### 1.jackson.jar+@ResponseBody

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
web场景自动引入了json场景
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-json</artifactId>
    <version>2.3.4.RELEASE</version>
    <scope>compile</scope>
</dependency>
```

##### 返回值解析器

![image-20210824091058373](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210824091058373.png)

```java
try {
    this.returnValueHandlers.handleReturnValue(
        returnValue, getReturnValueType(returnValue), mavContainer, webRequest);
}
```

```java
@Override
public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
                              ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception {

    HandlerMethodReturnValueHandler handler = selectHandler(returnValue, returnType);
    if (handler == null) {
        throw new IllegalArgumentException("Unknown return value type: " + returnType.getParameterType().getName());
    }
    handler.handleReturnValue(returnValue, returnType, mavContainer, webRequest);
}
```

```java
//RequestResponseBodyMethodProcessor  	
@Override
	public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
			ModelAndViewContainer mavContainer, NativeWebRequest webRequest)
			throws IOException, HttpMediaTypeNotAcceptableException, HttpMessageNotWritableException {

		mavContainer.setRequestHandled(true);
		ServletServerHttpRequest inputMessage = createInputMessage(webRequest);
		ServletServerHttpResponse outputMessage = createOutputMessage(webRequest);

		// Try even with null return value. ResponseBodyAdvice could get involved.
        // 使用消息转换器进行写出操作
		writeWithMessageConverters(returnValue, returnType, inputMessage, outputMessage);
	}
```

###### 返回值解析器原理

1. 返回值处理器判断是否支持这种类型返回值 supportsReturnType

2. 返回值处理器调用 handleReturnValue 进行处理

3. RequestResponseBodyMethodProcessor 可以处理返回值标了@ResponseBody 注解的。
   1. 利用 MessageConverters 进行处理 将数据写为json
      1. 内容协商（浏览器默认会以请求头的方式告诉服务器他能接受什么样的内容类型）
      2. 服务器最终根据自己自身的能力，决定服务器能生产出什么样内容类型的数据，
      3. SpringMVC会挨个遍历所有容器底层的 HttpMessageConverter ，看谁能处理？
         1. 得到MappingJackson2HttpMessageConverter可以将对象写为json
         2. 利用MappingJackson2HttpMessageConverter将对象转为json再写出去。



###### SpringMVC支持哪些返回值

```java
ModelAndView
Model
View
ResponseEntity 
ResponseBodyEmitter
StreamingResponseBody
HttpEntity
HttpHeaders
Callable
DeferredResult
ListenableFuture
CompletionStage
WebAsyncTask
有 @ModelAttribute 且为对象类型的
@ResponseBody 注解 ---> RequestResponseBodyMethodProcessor；

```

###### HTTPMessageConverter原理

![image-20210824093024306](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210824093024306.png)

HttpMessageConverter: 看是否支持将 此 Class类型的对象，转为MediaType类型的数据。

例子：Person对象转为JSON。或者 JSON转为Person

###### 默认的MessageConverter

![image-20210824093417501](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210824093417501.png)

0 - 只支持Byte类型的

1 - String

2 - String

3 - Resource

4 - ResourceRegion

5 - DOMSource.**class \** SAXSource.**class**) \ StAXSource.**class \**StreamSource.**class \**Source.**class**

**6 -** MultiValueMap

7 - **true** 

**8 - true**

**9 - 支持注解方式xml处理的**



最终 MappingJackson2HttpMessageConverter  把对象转为JSON（利用底层的jackson的objectMapper转换的）



##### 内容协商

根据客户端接收能力不同，返回不同媒体类型的数据。

###### 引入xml依赖

```xml
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```

###### postman分别测试返回json和xml

只需要改变请求头中Accept字段。Http协议中规定的，告诉服务器本客户端可以接收的数据类型。





###### 内容协商原理

1. 判断当前响应头中是否已经有确定的媒体类型。MediaType
2. **获取客户端（PostMan、浏览器）支持接收的内容类型。（获取客户端Accept请求头字段）【application/xml】**
   * **contentNegotiationManager 内容协商管理器 默认使用基于请求头的策略**
   * **HeaderContentNegotiationStrategy  确定客户端可以接收的内容类型** 

3. 遍历循环所有当前系统的 **MessageConverter**，看谁支持操作这个对象（Person）

4. 找到支持操作Person的converter，把converter支持的媒体类型统计出来。

5. 客户端需要【application/xml】。服务端能力【10种、json、xml】

6. 进行内容协商的最佳匹配媒体类型

7. 用 支持 将对象转为 最佳匹配媒体类型 的converter。调用它进行转化 。



导入了jackson处理xml的包，xml的converter就会自动进来

```java
WebMvcConfigurationSupport
    jackson2XmlPresent = ClassUtils.isPresent("com.fasterxml.jackson.dataformat.xml.XmlMapper", classLoader);

if (jackson2XmlPresent) {
    Jackson2ObjectMapperBuilder builder = Jackson2ObjectMapperBuilder.xml();
    if (this.applicationContext != null) {
        builder.applicationContext(this.applicationContext);
    }
    messageConverters.add(new MappingJackson2XmlHttpMessageConverter(builder.build()));
}
```



###### 开启浏览器参数方式内容协商功能

为了方便内容协商，开启基于请求参数的内容协商功能。

```yaml
spring:
    contentnegotiation:
      favor-parameter: true  #开启请求参数内容协商模式
```

```
发请求： http://localhost:8080/test/person?format=json
http://localhost:8080/test/person?format=xml
```

![image-20210824100112825](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210824100112825.png)

确定客户端接收什么样的内容类型；

1. Parameter策略优先确定是要返回json数据（获取请求头中的format的值）

2. 最终进行内容协商返回给客户端json即可。

###### 自定义 MessageConverter

**实现多协议数据兼容。json、xml、x-guigu**

**0、**@ResponseBody 响应数据出去 调用 **RequestResponseBodyMethodProcessor** 处理

1、Processor 处理方法返回值。通过 **MessageConverter** 处理

2、所有 **MessageConverter** 合起来可以支持各种媒体类型数据的操作（读、写）

3、内容协商找到最终的 **messageConverter**；



SpringMVC的什么功能。一个入口给容器中添加一个  WebMvcConfigurer

```java
@Bean
public WebMvcConfigurer webMvcConfigurer(){
    return new WebMvcConfigurer() {
        @Override
        public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
            converters.add(new GuiguMessageConverter());
        }
    }
}
```

```java
// 自定义的Converter
public class GuiguMessageConverter implements HttpMessageConverter<Person> {

    @Override
    public boolean canRead(Class<?> clazz, MediaType mediaType) {
        return false;
    }

    @Override
    public boolean canWrite(Class<?> clazz, MediaType mediaType) {
        return clazz.isAssignableFrom(Person.class);
    }

    /**
     * 服务器要统计所有MessageConverter都能写出哪些内容类型
     *
     * application/x-guigu
     * @return
     */
    @Override
    public List<MediaType> getSupportedMediaTypes() {
        return MediaType.parseMediaTypes("application/x-guigu");
    }

    @Override
    public Person read(Class<? extends Person> clazz, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException {
        return null;
    }

    @Override
    public void write(Person person, MediaType contentType, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException {
        //自定义协议数据的写出
        String data = person.getUserName()+";"+person.getAge()+";"+person.getBirth();


        //写出去
        OutputStream body = outputMessage.getBody();
        body.write(data.getBytes());
    }
}

```

**参数方式**

```java
//1、WebMvcConfigurer定制化SpringMVC的功能
@Bean
public WebMvcConfigurer webMvcConfigurer(){
    return new WebMvcConfigurer() {

        /**
             * 自定义内容协商策略
             * @param configurer
             */
        @Override
        public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
            //Map<String, MediaType> mediaTypes
            Map<String, MediaType> mediaTypes = new HashMap<>();
            mediaTypes.put("json",MediaType.APPLICATION_JSON);
            mediaTypes.put("xml",MediaType.APPLICATION_XML);
            mediaTypes.put("gg",MediaType.parseMediaType("application/x-guigu"));
            //指定支持解析哪些参数对应的哪些媒体类型
            ParameterContentNegotiationStrategy parameterStrategy = new ParameterContentNegotiationStrategy(mediaTypes);
            //                parameterStrategy.setParameterName("ff");

            HeaderContentNegotiationStrategy headeStrategy = new HeaderContentNegotiationStrategy();

            configurer.strategies(Arrays.asList(parameterStrategy,headeStrategy));
        }
```

**有可能我们添加的自定义的功能会覆盖默认很多功能，导致一些默认的功能失效。**

```yaml
spring:
  mvc:
    contentnegotiation:
      media-types: application/gg
spring.mvc.contentnegotiation.media-types.gg=application/x-guigu
```

#### 视图解析与模板引擎

视图解析：**SpringBoot默认不支持 JSP，需要引入第三方模板引擎技术实现页面渲染。**

##### 视图解析

###### 视图解析原理流程

1、目标方法处理的过程中，所有数据都会被放在 **ModelAndViewContainer 里面。包括数据和视图地址**

**2、方法的参数是一个自定义类型对象（从请求参数中确定的），把他重新放在** **ModelAndViewContainer** 

**3、任何目标方法执行完成以后都会返回 ModelAndView（数据和视图地址）。**

**4、processDispatchResult  处理派发结果（页面改如何响应）**

- 1、**render**(**mv**, request, response); 进行页面渲染逻辑
  - 根据方法的String返回值得到 **View** 对象【定义了页面的渲染逻辑】
    - 1、所有的视图解析器尝试是否能根据当前返回值得到**View**对象
    - 2、得到了  **redirect:/main.html** --> Thymeleaf new **RedirectView**()
    - 3、ContentNegotiationViewResolver 里面包含了下面所有的视图解析器，内部还是利用下面所有视图解析器得到视图对象。
    - 4、view.render(mv.getModelInternal(), request, response);   视图对象调用自定义的render进行页面渲染工作
      - **RedirectView 如何渲染【重定向到一个页面】**
      - **1、获取目标url地址**
      - **2、response.sendRedirect(encodedURL);**

**视图解析：**

- **返回值以 forward: 开始： **

  **new InternalResourceView(forwardUrl); -->  转发**

  **request.getRequestDispatcher(path).forward(request, response);** 

- **返回值以** **redirect: 开始：** **new RedirectView() --》 render就是重定向** 

- **返回值是普通字符串： new ThymeleafView（）--->** 

自定义视图解析器+自定义视图；

![image-20210824132410158](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210824132410158.png)

##### 模板引擎-Thymeleaf

**现代化、服务端Java模板引擎**

###### 基本语法

1. **表达式**

| 表达式名字 | 语法   | 用途                               |
| ---------- | ------ | ---------------------------------- |
| 变量取值   | ${...} | 获取请求域、session域、对象等值    |
| 选择变量   | *{...} | 获取上下文对象值                   |
| 消息       | #{...} | 获取国际化等值                     |
| 链接       | @{...} | 生成链接                           |
| 片段表达式 | ~{...} | jsp:include 作用，引入公共页面片段 |

2. **字面量**

文本值: **'one text'** **,** **'Another one!'** **,…**数字: **0** **,** **34** **,** **3.0** **,** **12.3** **,…**布尔值: **true** **,** **false**

空值: **null**

变量： one，two，.... 变量不能有空格

3. **文本操作**

字符串拼接: **+**

变量替换: **|The name is ${name}|** 

4. **数学运算**

运算符: + , - , * , / , %

5. **布尔运算**

运算符:  **and** **,** **or**

一元运算: **!** **,** **not** 

6. **比较运算**

比较: **>** **,** **<** **,** **>=** **,** **<=** **(** **gt** **,** **lt** **,** **ge** **,** **le** **)**等式: **==** **,** **!=** **(** **eq** **,** **ne** **)** 

7. **条件运算**

If-then: **(if) ? (then)**

If-then-else: **(if) ? (then) : (else)**

Default: (value) **?: (defaultvalue)** 

8. **特殊操作**

无操作： _

###### 设置属性值-th:attr

设置单个值

```html
<form action="subscribe.html" th:attr="action=@{/subscribe}">
  <fieldset>
    <input type="text" name="email" />
    <input type="submit" value="Subscribe!" th:attr="value=#{subscribe.submit}"/>
  </fieldset>
</form>
```

设置多个值

```html
<img src="../../images/gtvglogo.png"  th:attr="src=@{/images/gtvglogo.png},title=#{logo},alt=#{logo}" />
```

以上两个的代替写法 th:xxxx

```html
<input type="submit" value="Subscribe!" th:value="#{subscribe.submit}"/>
<form action="subscribe.html" th:action="@{/subscribe}">
```

所有h5兼容的标签写法

https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#setting-value-to-specific-attributes

###### 迭代

```html
<tr th:each="prod : ${prods}">
        <td th:text="${prod.name}">Onions</td>
        <td th:text="${prod.price}">2.41</td>
        <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
</tr>
```

```html
<tr th:each="prod,iterStat : ${prods}" th:class="${iterStat.odd}? 'odd'">
  <td th:text="${prod.name}">Onions</td>
  <td th:text="${prod.price}">2.41</td>
  <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
</tr>
```

###### 条件运算

```html
<a href="comments.html"
th:href="@{/product/comments(prodId=${prod.id})}"
th:if="${not #lists.isEmpty(prod.comments)}">view</a>
```

```html
<div th:switch="${user.role}">
  <p th:case="'admin'">User is an administrator</p>
  <p th:case="#{roles.manager}">User is a manager</p>
  <p th:case="*">User is some other thing</p>
</div>
```

###### 属性优先级

![image-20210824105747490](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210824105747490.png)



##### thymeleaf使用

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

自动配置好了thymeleaf

```java
@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(ThymeleafProperties.class)
@ConditionalOnClass({ TemplateMode.class, SpringTemplateEngine.class })
@AutoConfigureAfter({ WebMvcAutoConfiguration.class, WebFluxAutoConfiguration.class })
public class ThymeleafAutoConfiguration { }
```

自动配好的策略

- 1、所有thymeleaf的配置值都在 ThymeleafProperties
- 2、配置好了 **SpringTemplateEngine** 

- **3、配好了** **ThymeleafViewResolver** 
- 4、我们只需要直接开发页面

```java
public static final String DEFAULT_PREFIX = "classpath:/templates/";

public static final String DEFAULT_SUFFIX = ".html";  //xxx.html
```

###### 页面开发

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1 th:text="${msg}">哈哈</h1>
<h2>
    <a href="www.atguigu.com" th:href="${link}">去百度</a>  <br/>
    <a href="www.atguigu.com" th:href="@{link}">去百度2</a>
</h2>
</body>
</html>
```

```java
@Controller
public class ViewTestController {
    @GetMapping("/atguigu")
    public String atguigu(Model model){
        //model中的数据会被放在请求域中 request.setAttribute("a",aa)
        model.addAttribute("msg","你好 guigu");
        model.addAttribute("link","http://www.baidu.com");
        return "success";
    }
}
```

##### 构建后台管理系统

###### 1、项目创建

thymeleaf、web-starter、devtools、lombok

###### 2、静态资源处理

自动配置好，我们只需要把所有静态资源放到 static 文件夹下

###### 3、路径构建

th:action="@{/login}"

###### 4、模板抽取

th:insert/replace/include

###### 5、页面跳转

```java
@PostMapping("/login")
public String main(User user, HttpSession session, Model model){

    if(StringUtils.hasLength(user.getUserName()) && "123456".equals(user.getPassword())){
        //把登陆成功的用户保存起来
        session.setAttribute("loginUser",user);
        //登录成功重定向到main.html;  重定向防止表单重复提交
        return "redirect:/main.html";
    }else {
        model.addAttribute("msg","账号密码错误");
        //回到登录页面
        return "login";
    }

}
```

###### 6、数据渲染

```java
    @GetMapping("/dynamic_table")
    public String dynamic_table(Model model){
        //表格内容的遍历
        List<User> users = Arrays.asList(new User("zhangsan", "123456"),
                new User("lisi", "123444"),
                new User("haha", "aaaaa"),
                new User("hehe ", "aaddd"));
        model.addAttribute("users",users);

        return "table/dynamic_table";
    }
```

```html
        <table class="display table table-bordered" id="hidden-table-info">
        <thead>
        <tr>
            <th>#</th>
            <th>用户名</th>
            <th>密码</th>
        </tr>
        </thead>
        <tbody>
        <tr class="gradeX" th:each="user,stats:${users}">
            <td th:text="${stats.count}">Trident</td>
            <td th:text="${user.userName}">Internet</td>
            <td >[[${user.password}]]</td>
        </tr>
        </tbody>
        </table>
```

#### 拦截器

##### HandlerInterceptor 接口

```java
/**
 * 登录检查
 * 1、配置好拦截器要拦截哪些请求
 * 2、把这些配置放在容器中
 */
@Slf4j
public class LoginInterceptor implements HandlerInterceptor {
    /**
     * 目标方法执行之前
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        String requestURI = request.getRequestURI();
        log.info("preHandle拦截的请求路径是{}",requestURI);

        //登录检查逻辑
        HttpSession session = request.getSession();
        Object loginUser = session.getAttribute("loginUser");
        if(loginUser != null){
            //放行
            return true;
        }
        //拦截住。未登录。跳转到登录页
        request.setAttribute("msg","请先登录");
//        re.sendRedirect("/");
        request.getRequestDispatcher("/").forward(request,response);
        return false;
    }
    /**
     * 目标方法执行完成以后
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        log.info("postHandle执行{}",modelAndView);
    }
    /**
     * 页面渲染以后
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        log.info("afterCompletion执行异常{}",ex);
    }
}
```

##### 配置拦截器

```java
/**
 * 1、编写一个拦截器实现HandlerInterceptor接口
 * 2、拦截器注册到容器中（实现WebMvcConfigurer的addInterceptors）
 * 3、指定拦截规则【如果是拦截所有，静态资源也会被拦截】
 */
@Configuration
public class AdminWebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor())
                .addPathPatterns("/**")  //所有请求都被拦截包括静态资源
                .excludePathPatterns("/","/login","/css/**","/fonts/**","/images/**","/js/**"); //放行的请求
    }
}
```

##### 拦截器原理

1、根据当前请求，找到**HandlerExecutionChain【**可以处理请求的handler以及handler的所有 拦截器】

2、先来**顺序执行** 所有拦截器的 preHandle方法

- 1、如果当前拦截器prehandler返回为true。则执行下一个拦截器的preHandle
- 2、如果当前拦截器返回为false。直接    倒序执行所有已经执行了的拦截器的  afterCompletion；

**3、如果任何一个拦截器返回false。直接跳出不执行目标方法**

**4、所有拦截器都返回True。执行目标方法**

**5、倒序执行所有拦截器的postHandle方法。**

**6、前面的步骤有任何异常都会直接倒序触发** afterCompletion

7、页面成功渲染完成以后，也会倒序触发 afterCompletion

![image-20210824133652656](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210824133652656.png)

![image-20210824134132228](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210824134132228.png)

#### 文件上传

##### 页面表单

multiple多文件

```html
<form method="post" action="/upload" enctype="multipart/form-data">
    <input type="file" name="file" multiple><br>
    <input type="submit" value="提交">
</form>
```

##### 文件上传

```java
    /**
     * MultipartFile 自动封装上传过来的文件
     */
    @PostMapping("/upload")
    public String upload(@RequestParam("email") String email,
                         @RequestParam("username") String username,
                         @RequestPart("headerImg") MultipartFile headerImg,
                         @RequestPart("photos") MultipartFile[] photos) throws IOException {

        log.info("上传的信息：email={}，username={}，headerImg={}，photos={}",
                email,username,headerImg.getSize(),photos.length);

        if(!headerImg.isEmpty()){
            //保存到文件服务器，OSS服务器
            String originalFilename = headerImg.getOriginalFilename();
            headerImg.transferTo(new File("H:\\cache\\"+originalFilename));
        }
        if(photos.length > 0){
            for (MultipartFile photo : photos) {
                if(!photo.isEmpty()){
                    String originalFilename = photo.getOriginalFilename();
                    photo.transferTo(new File("H:\\cache\\"+originalFilename));
                }
            }
        }
        return "main";
    }
```

##### 自动配置原理

**文件上传自动配置类-MultipartAutoConfiguration-MultipartProperties**

- 自动配置好了 **StandardServletMultipartResolver   【文件上传解析器】**
- **原理步骤**
  - **1、请求进来使用文件上传解析器判断（**isMultipart**）并封装（**resolveMultipart，**返回**MultipartHttpServletRequest**）文件上传请求**
  - **2、参数解析器来解析请求中的文件内容封装成MultipartFile**
  - **3、将request中文件信息封装为一个Map；**MultiValueMap<String, MultipartFile>

**FileCopyUtils**。实现文件流的拷贝

```java
@PostMapping("/upload")
public String upload(@RequestParam("email") String email,
                     @RequestParam("username") String username,
                     @RequestPart("headerImg") MultipartFile headerImg,
                     @RequestPart("photos") MultipartFile[] photos)
```

#### 异常处理

##### 默认规则

- 默认情况下，Spring Boot提供`/error`处理所有错误的映射

- 对于机器客户端，它将生成JSON响应，其中包含错误，HTTP状态和异常消息的详细信息。

  对于浏览器客户端，响应一个“ whitelabel”错误视图，以HTML格式呈现相同的数据

- **要对其进行自定义，添加View解析为error**
- 要完全替换默认行为，可以实现 `ErrorController `并注册该类型的Bean定义，或添加`ErrorAttributes类型的组件`以使用现有机制但替换其内容。

- error/下的4xx，5xx页面会被自动解析；

##### 自定义错误处理逻辑

- 自定义错误页

- - error/404.html   error/5xx.html；有精确的错误状态码页面就匹配精确，没有就找 4xx.html；如果都没有就触发白页

- @ControllerAdvice+@ExceptionHandler处理全局异常；

  底层是系统默认的  异常解析器 **ExceptionHandlerExceptionResolver 支持的**

  ```java
  @ControllerAdvice
  public class GlobalExceptionHandler {
      @ExceptionHandler({ArithmeticException.class,NullPointerException.class})  //处理异常
      public String handleArithException(Exception e){
          log.error("异常是：{}",e);
          return "login"; //视图地址
      }
  }
  ```

- @ResponseStatus+自定义异常 ；

  底层是系统默认的 **ResponseStatusExceptionResolver ，把responsestatus注解的信息底层调用** **response.sendError(statusCode, resolvedReason)；tomcat发送的/error**

  ```java
  @ResponseStatus(value= HttpStatus.FORBIDDEN,reason = "用户数量太多")
  public class UserTooManyException extends RuntimeException {
      public  UserTooManyException(){}
      public  UserTooManyException(String message){
          super(message);
      }
  }
  ```

- Spring底层的异常，如 参数类型转换异常；**DefaultHandlerExceptionResolver 处理框架底层的异常。**

- - response.sendError(HttpServletResponse.**SC_BAD_REQUEST**, ex.getMessage()); 

- 自定义实现 HandlerExceptionResolver 处理异常；可以作为默认的全局异常处理规则

  ```java
  @Order(value= Ordered.HIGHEST_PRECEDENCE)  //优先级，数字越小优先级越高
  @Component
  public class CustomerHandlerExceptionResolver implements HandlerExceptionResolver {
      @Override
      public ModelAndView resolveException(HttpServletRequest request,
                                           HttpServletResponse response,
                                           Object handler, Exception ex) {
          try {
              response.sendError(511,"我喜欢的错误");
          } catch (IOException e) {
              e.printStackTrace();
          }
          return new ModelAndView();
      }
  }
  ```

* **ErrorViewResolver**  实现自定义处理异常；

  * response.sendError 。error请求就会转给controller
  * 你的异常没有任何人能处理。tomcat底层 response.sendError。error请求就会转给controller
  * **basicErrorController 要去的页面地址是** **ErrorViewResolver**  

##### 异常处理自动配置原理

**ErrorMvcAutoConfiguration  自动配置异常处理规则**

* **容器中的组件：类型：DefaultErrorAttributes ->** **id：errorAttributes**

  **public class** **DefaultErrorAttributes** **implements** **ErrorAttributes**, **HandlerExceptionResolver**

  定义错误页面中可以包含哪些数据

  ```java
  public Map<String, Object> getErrorAttributes(ServerRequest request, ErrorAttributeOptions options) {
      Map<String, Object> errorAttributes = this.getErrorAttributes(request, options.isIncluded(Include.STACK_TRACE));
      if (Boolean.TRUE.equals(this.includeException)) {
          options = options.including(new Include[]{Include.EXCEPTION});
      }
      if (!options.isIncluded(Include.EXCEPTION)) {
          errorAttributes.remove("exception");
      }
      if (!options.isIncluded(Include.STACK_TRACE)) {
          errorAttributes.remove("trace");
      }
      if (!options.isIncluded(Include.MESSAGE) && errorAttributes.get("message") != null){
          errorAttributes.put("message", "");
      }
      if (!options.isIncluded(Include.BINDING_ERRORS)) {
          errorAttributes.remove("errors");
      }
      return errorAttributes;
  }
  ```

* **容器中的组件：类型：BasicErrorController --> id：basicErrorController（json+白页 适配响应）**
   * **处理默认** **/error 路径的请求；页面响应** **new** ModelAndView(**"error"**, model)；
  * **容器中有组件 View**->**id是error**；（响应默认错误页）

   * 容器中放组件 **BeanNameViewResolver（视图解析器）；按照返回的视图名作为组件的id去容器中找View对象。**

* **容器中的组件：**类型：**DefaultErrorViewResolver -> id：**conventionErrorViewResolver
   * 如果发生错误，会以HTTP的状态码 作为视图页地址（viewName），找到真正的页面
  * error/404、5xx.html

```java
//BasicErrorController
@RequestMapping(produces = {"text/html"})
public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response)//返回html
@RequestMapping
public ResponseEntity<Map<String, Object>> error(HttpServletRequest request)//返回json
```

##### 异常处理步骤流程

1. 执行目标方法，目标方法运行期间有任何异常都会被catch、而且标志当前请求结束；并且用 **dispatchException** 

2. 进入视图解析流程（页面渲染？）

   processDispatchResult(processedRequest, response, mappedHandler, **mv**, **dispatchException**);

3. **mv** = **processHandlerException**；处理handler发生的异常，处理完成返回ModelAndView；

   1. 遍历所有的 **handlerExceptionResolvers，看谁能处理当前异常【HandlerExceptionResolver处理器异常解析器】**

   2. 系统默认的  异常解析器；

      ![image-20210826152058944](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210826152058944.png)

      1. **DefaultErrorAttributes先来处理异常。把异常信息保存到request域，并且返回null；**
      2. **默认没有任何人能处理异常，所以异常会被抛出**
         1. **如果没有任何人能处理最终底层就会发送 /error 请求。会被底层的BasicErrorController处理**
         2. **解析错误视图；遍历所有的**  **ErrorViewResolver  看谁能解析。**
         3. **默认的** **DefaultErrorViewResolver ,作用是把响应状态码作为错误页的地址，error/500.html** 
         4. **模板引擎最终响应这个页面** **error/500.html**

#### Web原生组件注入Servlet/Filter/Listener

##### 使用Servlet API

```java
@ServletComponentScan(basePackages = "com.atguigu.admin") //指定原生Servlet组件都放在那里
@WebServlet(urlPatterns = "/my")//效果：直接响应，没有经过Spring的拦截器？
@WebFilter(urlPatterns={"/css/*","/images/*"})
@WebListener
```

```java
@Slf4j
@WebFilter(urlPatterns={"/css/*","/images/*"}) //my
public class MyFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        log.info("MyFilter初始化完成");
    }
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        log.info("MyFilter工作");
        chain.doFilter(request,response);
    }
    @Override
    public void destroy() {
        log.info("MyFilter销毁");
    }
}
@WebServlet(urlPatterns = "/my")
public class MyServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().write("66666");
    }
}
@Slf4j
@WebListener
public class MySwervletContextListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        log.info("MySwervletContextListener监听到项目初始化完成");
    }
    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        log.info("MySwervletContextListener监听到项目销毁");
    }
}
```

扩展：DispatchServlet 如何注册进来

- 容器中自动配置了  DispatcherServlet  属性绑定到 WebMvcProperties；对应的配置文件配置项是 **spring.mvc。**
- **通过** **ServletRegistrationBean**<DispatcherServlet> 把 DispatcherServlet  配置进来。

- 默认映射的是 / 路径。



Tomcat-Servlet；

多个Servlet都能处理到同一层路径，精确优选原则

A： /my/

B： /my/1

##### 使用RegistrationBean

```
ServletRegistrationBean
FilterRegistrationBean
ServletListenerRegistrationBean
```

```java
// (proxyBeanMethods = true)：保证依赖的组件始终是单实例的
@Configuration(proxyBeanMethods = true)
public class MyRegistConfig {
    @Bean
    public ServletRegistrationBean myServlet(){
        MyServlet myServlet = new MyServlet();
        return new ServletRegistrationBean(myServlet,"/my","/my02");
    }
    @Bean
    public FilterRegistrationBean myFilter(){
        MyFilter myFilter = new MyFilter();
//        return new FilterRegistrationBean(myFilter,myServlet());
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean(myFilter);
        filterRegistrationBean.setUrlPatterns(Arrays.asList("/my","/css/*"));
        return filterRegistrationBean;
    }
    @Bean
    public ServletListenerRegistrationBean myListener(){
        MySwervletContextListener mySwervletContextListener = new MySwervletContextListener();
        return new ServletListenerRegistrationBean(mySwervletContextListener);
    }
}
```

#### 嵌入式Servlet容器

##### 切换嵌入式Servlet容器

- 默认支持的webServer

- - `Tomcat`, `Jetty`, or `Undertow` `netty`
  - `ServletWebServerApplicationContext 容器启动寻找ServletWebServerFactory 并引导创建服务器`

- 切换服务器

  排除tomcat

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <exclusions>
          <exclusion>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-tomcat</artifactId>
          </exclusion>
      </exclusions>
  </dependency>
  ```

- 原理

- - SpringBoot应用启动发现当前是Web应用。web场景包-导入tomcat
  - web应用会创建一个web版的ioc容器 `ServletWebServerApplicationContext` 

- - `ServletWebServerApplicationContext` 启动的时候寻找

     **ServletWebServerFactory**``（Servlet 的web服务器工厂---> Servlet 的web服务器）

  - SpringBoot底层默认有很多的WebServer工厂

    `TomcatServletWebServerFactory`

    `JettyServletWebServerFactory`

     `UndertowServletWebServerFactory`

- - 底层直接会有一个自动配置类。`ServletWebServerFactoryAutoConfiguration`
  - ServletWebServerFactoryAutoConfiguration导入了ServletWebServerFactoryConfiguration（配置类）

- - ServletWebServerFactoryConfiguration 配置类 根据动态判断系统中到底导入了那个Web服务器的包。（默认是web-starter导入tomcat包），容器中就有 TomcatServletWebServerFactory
  - TomcatServletWebServerFactory 创建出Tomcat服务器并启动；TomcatWebServer 的构造器拥有初始化方法initialize---this.tomcat.start();

- - 内嵌服务器，就是手动把启动服务器的代码调用（tomcat核心jar包存在

##### 定制Servlet容器

- 实现  **WebServerFactoryCu**stomizer<ConfigurableServletWebServerFactory> 

- - 把配置文件的值和**ServletWebServerFactory 进行绑定**

- 修改配置文件 **server.xxx**
- 直接自定义 **ConfigurableServletWebServerFactory** 



**xxxxxCustomizer：定制化器，可以改变xxxx的默认规则**

```java
import org.springframework.boot.web.server.WebServerFactoryCustomizer;
import org.springframework.boot.web.servlet.server.ConfigurableServletWebServerFactory;
import org.springframework.stereotype.Component;

@Component
public class CustomizationBean implements 
    			WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> {
    @Override
    public void customize(ConfigurableServletWebServerFactory server) {
        server.setPort(9000);
    }
}
```

#### 定制化原理

##### 定制化的常见方式 

1. 修改配置文件；
2. **xxxxxCustomizer；**
3. **编写自定义的配置类   xxxConfiguration；+** **@Bean替换、增加容器中默认组件；视图解析器** 
4. **Web应用 编写一个配置类实现** **WebMvcConfigurer 即可定制化web功能；+ @Bean给容器中再扩展一些组件**

```java
@Configuration
public class AdminWebConfig implements WebMvcConfigurer
```

5. @EnableWebMvc + WebMvcConfigurer —— @Bean  可以全面接管SpringMVC，所有规则全部自己重新配置； 实现定制和扩展功能



 @EnableWebMvc原理

1. WebMvcAutoConfiguration  默认的SpringMVC的自动配置功能类。静态资源、欢迎页.....

2. 一旦使用 @EnableWebMvc 会 @Import(DelegatingWebMvcConfiguration.**class**)
3. **DelegatingWebMvcConfiguration** 的 作用，只保证SpringMVC最基本的使用 
   * 把所有系统中的 WebMvcConfigurer 拿过来。所有功能的定制都是这些 WebMvcConfigurer  合起来一起生效

   * 自动配置了一些非常底层的组件。**RequestMappingHandlerMapping**、这些组件依赖的组件都是从容器中获取
   * **public class** DelegatingWebMvcConfiguration **extends** **WebMvcConfigurationSupport**

4. **WebMvcAutoConfiguration** 里面的配置要能生效 必须

   @ConditionalOnMissingBean(**WebMvcConfigurationSupport**.**class**)、

5. @EnableWebMvc  导致了 **WebMvcAutoConfiguration  没有生效。**

##### 原理分析套路

**场景starter** **- xxxxAutoConfiguration - 导入xxx组件 - 绑定xxxProperties -- 修改配置文件项**

### 数据访问

#### SQL

##### 导入JDBC

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>
```

数据库版本和驱动版本对应

想要修改版本

1. 直接依赖引入具体版本（maven的就近依赖原则）
2. 重新声明版本（maven的属性的就近优先原则）

```xml
默认版本：<mysql.version>8.0.22</mysql.version>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <!--            <version>5.1.49</version>-->
</dependency>
<properties>
    <java.version>1.8</java.version>
    <mysql.version>5.1.49</mysql.version>
</properties>
```

##### 分析自动配置

- DataSourceAutoConfiguration ： 数据源的自动配置

- - 修改数据源相关的配置：**spring.datasource**
  - **数据库连接池的配置，是自己容器中没有DataSource才自动配置的**

- - 底层配置好的连接池是：**HikariDataSource**

    ```java
    @Configuration(proxyBeanMethods = false)
    @Conditional(PooledDataSourceCondition.class)
    @ConditionalOnMissingBean({ DataSource.class, XADataSource.class })
    @Import({ DataSourceConfiguration.Hikari.class, DataSourceConfiguration.Tomcat.class,
             DataSourceConfiguration.Dbcp2.class, DataSourceConfiguration.OracleUcp.class,
             DataSourceConfiguration.Generic.class, DataSourceJmxConfiguration.class })
    protected static class PooledDataSourceConfiguration
    ```

- DataSourceTransactionManagerAutoConfiguration： 事务管理器的自动配置
- JdbcTemplateAutoConfiguration： **JdbcTemplate的自动配置，可以来对数据库进行crud**

- - 可以修改这个配置项@ConfigurationProperties(prefix = **"spring.jdbc"**) 来修改JdbcTemplate
  - @Bean@Primary    JdbcTemplate；容器中有这个组件

- JndiDataSourceAutoConfiguration： jndi的自动配置
- XADataSourceAutoConfiguration： 分布式事务相关的

##### 修改配置项

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/db_account
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
```

##### 测试

```java
@Slf4j
@SpringBootTest
class Boot05WebAdminApplicationTests {

    @Autowired
    JdbcTemplate jdbcTemplate;


    @Test
    void contextLoads() {

//        jdbcTemplate.queryForObject("select * from account_tbl")
//        jdbcTemplate.queryForList("select * from account_tbl",)
        Long aLong = jdbcTemplate.queryForObject("select count(*) from account_tbl", Long.class);
        log.info("记录总数：{}",aLong);
    }

}
```

##### 使用Druid

https://github.com/alibaba/druid

###### 自定义方式

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.17</version>
</dependency>
```

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/db_account
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
    
    druid:
      aop-patterns: com.atguigu.admin.*  #springbean监控
      filters: stat,wall,slf4j  #所有开启的功能

      stat-view-servlet:  #监控页配置
        enabled: true
        login-username: admin
        login-password: admin
        resetEnable: false

      web-stat-filter:  #web监控
        enabled: true
        urlPattern: /*
        exclusions: '*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*'


      filter:
        stat: #sql监控
          slow-sql-millis: 1000 #慢sql监控
          logSlowSql: true
          enabled: true
        wall: #防火墙
          enabled: true
          config:
            drop-table-allow: false
```

或者

```java
@Configuration
public class MyDataSourceConfig {

    // 默认的自动配置是判断容器中没有才会配@ConditionalOnMissingBean(DataSource.class)
    @ConfigurationProperties("spring.datasource")
    @Bean
    public DataSource dataSource() throws SQLException {
        DruidDataSource druidDataSource = new DruidDataSource();
        //加入监控功能
        druidDataSource.setFilters("stat,wall");

        druidDataSource.setMaxActive(10);
        return druidDataSource;
    }

    /**
     * 配置 druid的监控页功能
     * @return
     */
    //    @Bean
    public ServletRegistrationBean statViewServlet(){
        StatViewServlet statViewServlet = new StatViewServlet();
        ServletRegistrationBean<StatViewServlet> registrationBean = new ServletRegistrationBean<>(statViewServlet, "/druid/*");

        registrationBean.addInitParameter("loginUsername","admin");
        registrationBean.addInitParameter("loginPassword","123456");


        return registrationBean;
    }

    /**
     * WebStatFilter 用于采集web-jdbc关联监控的数据。
     */
    //    @Bean
    public FilterRegistrationBean webStatFilter(){
        WebStatFilter webStatFilter = new WebStatFilter();

        FilterRegistrationBean<WebStatFilter> filterRegistrationBean = new FilterRegistrationBean<>(webStatFilter);
        filterRegistrationBean.setUrlPatterns(Arrays.asList("/*"));
        filterRegistrationBean.addInitParameter("exclusions","*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*");

        return filterRegistrationBean;
    }

}

```

###### starter方式

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.17</version>
</dependency>
```

分析自动配置

- 扩展配置项 **spring.datasource.druid**
- DruidSpringAopConfiguration.**class**,   监控SpringBean的；配置项：**spring.datasource.druid.aop-patterns**

- DruidStatViewServletConfiguration.**class**, 监控页的配置：**spring.datasource.druid.stat-view-servlet；默认开启**
-  DruidWebStatFilterConfiguration.**class**, web监控配置；**spring.datasource.druid.web-stat-filter；默认开启**

- DruidFilterConfiguration.**class**}) 所有Druid自己filter的配置

  ```java
  private static final String FILTER_STAT_PREFIX = "spring.datasource.druid.filter.stat";
  private static final String FILTER_CONFIG_PREFIX = "spring.datasource.druid.filter.config";
  private static final String FILTER_ENCODING_PREFIX = "spring.datasource.druid.filter.encoding";
  private static final String FILTER_SLF4J_PREFIX = "spring.datasource.druid.filter.slf4j";
  private static final String FILTER_LOG4J_PREFIX = "spring.datasource.druid.filter.log4j";
  private static final String FILTER_LOG4J2_PREFIX = "spring.datasource.druid.filter.log4j2";
  private static final String FILTER_COMMONS_LOG_PREFIX = "spring.datasource.druid.filter.commons-log";
  private static final String FILTER_WALL_PREFIX = "spring.datasource.druid.filter.wall";
  ```

配置示例

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/db_account
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver

    druid:
      aop-patterns: com.atguigu.admin.*  #监控SpringBean
      filters: stat,wall     # 底层开启功能，stat（sql监控），wall（防火墙）

      stat-view-servlet:   # 配置监控页功能
        enabled: true
        login-username: admin
        login-password: admin
        resetEnable: false

      web-stat-filter:  # 监控web
        enabled: true
        urlPattern: /*
        exclusions: '*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*'


      filter:
        stat:    # 对上面filters里面的stat的详细配置
          slow-sql-millis: 1000
          logSlowSql: true
          enabled: true
        wall:
          enabled: true
          config:
            drop-table-allow: false
```

SpringBoot配置示例

https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter

配置项列表

[https://github.com/alibaba/druid/wiki/DruidDataSource%E9%85%8D%E7%BD%AE%E5%B1%9E%E6%80%A7%E5%88%97%E8%A1%A8](https://github.com/alibaba/druid/wiki/DruidDataSource配置属性列表)

##### 整合MyBatis操作

https://github.com/mybatis

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.4</version>
</dependency>
```

###### 配置模式

- 全局配置文件
- SqlSessionFactory: 自动配置好了

- SqlSession：自动配置了 **SqlSessionTemplate 组合了SqlSession**
- @Import(**AutoConfiguredMapperScannerRegistrar**.**class**）；

- Mapper： 只要我们写的操作MyBatis的接口标准了 **@Mapper 就会被自动扫描进来**

```yaml
# 配置mybatis规则
mybatis:
  #config-location: classpath:mybatis/mybatis-config.xml  #全局配置文件位置
  mapper-locations: classpath:mybatis/mapper/*.xml  #sql映射文件位置
   configuration:
    map-underscore-to-camel-case: true
```

配置 **private** Configuration **configuration**; mybatis.**configuration下面的所有，就是相当于改mybatis全局配置文件中的值**

###### 注解模式/混合模式

```java
public interface CityMapper {
    @Select("select * from city where id=#{id}")
    public City getById(Long id);

    @Insert("insert into  city(`name`,`state`,`country`) values(#{name},#{state},#{country})")
    @Options(useGeneratedKeys = true,keyProperty = "id")
    public void insert(City city);

}
```

*@MapperScan("com.atguigu.admin.mapper") 简化，其他的接口就可以不用标注@Mapper注解*

##### 整合 MyBatis-Plus

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.1</version>
</dependency>
```

自动配置

- MybatisPlusAutoConfiguration 配置类，MybatisPlusProperties 配置项绑定。**mybatis-plus：xxx 就是对mybatis-plus的定制**
- **SqlSessionFactory 自动配置好。底层是容器中默认的数据源**

- **mapperLocations 自动配置好的。有默认值。*classpath\*:/mapper/\**/\*.xml；任意包的类路径下的所有mapper文件夹下任意路径下的所有xml都是sql映射文件。  建议以后sql映射文件，放在 mapper下**
- **容器中也自动配置好了** **SqlSessionTemplate**

- **@Mapper 标注的接口也会被自动扫描；建议直接** @MapperScan(**"com.atguigu.admin.mapper"**) 批量扫描就行

#### NOSQL

##### redis

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

自动配置：

- RedisAutoConfiguration 自动配置类。RedisProperties 属性类 --> **spring.redis.xxx是对redis的配置**
- 连接工厂是准备好的。**Lettuce**ConnectionConfiguration、**Jedis**ConnectionConfiguration

- **自动注入了RedisTemplate**<**Object**, **Object**> ： xxxTemplate；
- **自动注入了StringRedisTemplate；k：v都是String**

- **key：value**
- **底层只要我们使用** **StringRedisTemplate、RedisTemplate就可以操作redis**



**redis环境搭建**

**1、阿里云按量付费redis。经典网络**

**2、申请redis的公网连接地址**

**3、修改白名单  允许0.0.0.0/0 访问**

###### 配置

```yaml
  redis:
    host: r-bp1nc7reqesxisgxpipd.redis.rds.aliyuncs.com
    port: 6379
    password: lfy:Lfy123456
    client-type: jedis
    jedis:
      pool:
        max-active: 10
#    url: redis://lfy:Lfy123456@r-bp1nc7reqesxisgxpipd.redis.rds.aliyuncs.com:6379
#    lettuce:
#      pool:
#        max-active: 10
#        min-idle: 5
```

###### RedisTemplate与Lettuce

```java
@Test
void testRedis(){
    ValueOperations<String, String> operations = redisTemplate.opsForValue();

    operations.set("hello","world");

    String hello = operations.get("hello");
    System.out.println(hello);
}
```

###### 切换至jedis

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!--        导入jedis-->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
```

```yaml
spring:
  redis:
      host: r-bp1nc7reqesxisgxpipd.redis.rds.aliyuncs.com
      port: 6379
      password: lfy:Lfy123456
      client-type: jedis
      jedis:
        pool:
          max-active: 10
```

### 单元测试

#### JUnit5 的变化

**Spring Boot 2.2.0 版本开始引入 JUnit 5 作为单元测试默认库**

> **JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage**

**JUnit Platform**: Junit Platform是在JVM上启动测试框架的基础，不仅支持Junit自制的测试引擎，其他测试引擎也都可以接入。

**JUnit Jupiter**: JUnit Jupiter提供了JUnit5的新的编程模型，是JUnit5新特性的核心。内部 包含了一个**测试引擎**，用于在Junit Platform上运行。

**JUnit Vintage**: 由于JUint已经发展多年，为了照顾老的项目，JUnit Vintage提供了兼容JUnit4.x,Junit3.x的测试引擎。

注意：

**SpringBoot 2.4 以上版本移除了默认对** **Vintage 的依赖。如果需要兼容junit4需要自行引入（不能使用junit4的功能 @Test）**

**JUnit 5’s Vintage Engine Removed from** **spring-boot-starter-test,如果需要继续兼容junit4需要自行引入vintage**

```xml
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <scope>test</scope>
    <exclusions>
        <exclusion>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
</dependency>
```

现在版本

```java
@SpringBootTest
class Boot05WebAdminApplicationTests {
    @Test
    void contextLoads() {
    }
}
```

以前：

@SpringBootTest + @RunWith(SpringTest.class)

- Junit类具有Spring的功能，@Autowired、比如 @Transactional 标注测试方法，测试完成后自动回滚

#### 常用注解

JUnit5的注解与JUnit4的注解有所变化

https://junit.org/junit5/docs/current/user-guide/#writing-tests-annotations

```java
@Test //表示方法是测试方法。但是与JUnit4的@Test不同，他的职责非常单一不能声明任何属性，拓展的测试将会由Jupiter提供额外测试   import org.junit.jupiter.api.Test
@ParameterizedTest //表示方法是参数化测试，下方会有详细介绍
@RepeatedTest //表示方法可重复执行   @RepeatedTest(5)
@DisplayName //为测试类或者测试方法设置展示名称

@BeforeEach //表示在每个单元测试之前执行
@AfterEach //表示在每个单元测试之后执行
@BeforeAll //表示在所有单元测试之前执行
@AfterAll //表示在所有单元测试之后执行

@Tag //表示单元测试类别，类似于JUnit4中的@Categories
@Disabled //表示测试类或测试方法不执行，类似于JUnit4中的@Ignore
@Timeout //表示测试方法运行如果超过了指定时间将会返回错误  
//@Timeout(value = 500, unit = TimeUnit.MILLISECONDS)
@ExtendWith //为测试类或测试方法提供扩展类引用  @ExtendWith({SpringExtension.class})
```

#### 断言（assertions）

**这些断言方法都是 org.junit.jupiter.api.Assertions 的静态方法**。JUnit 5 内置的断言可以分成如下几个类别：

**检查业务逻辑返回的数据是否合理。**

**所有的测试运行结束以后，会有一个详细的测试报告；**



断言：前面断言失败，后面的代码都不会执行

##### 简单断言

```java
assertEquals	//判断两个对象或两个原始类型是否相等
assertNotEquals	//判断两个对象或两个原始类型是否不相等
assertSame		//判断两个对象引用是否指向同一个对象
assertNotSame	//判断两个对象引用是否指向不同的对象
assertTrue		//判断给定的布尔值是否为 true
assertFalse		//判断给定的布尔值是否为 false
assertNull		//判断给定的对象引用是否为 null
assertNotNull	//判断给定的对象引用是否不为 null
```

##### 数组断言

```java
assertArrayEquals
```

##### 组合断言

assertAll 方法接受多个 org.junit.jupiter.api.Executable 函数式接口的实例作为要验证的断言，可以通过 lambda 表达式很容易的提供这些断言

```java
 assertAll("test",
                () -> assertTrue(true && true, "结果不为true"),
                () -> assertEquals(1, 2, "结果不是1"));
```

##### 异常断言

**Assertions.assertThrows()** 

```java
@DisplayName("异常断言")
@Test
void testException() {
    //断定业务逻辑一定出现异常
    assertThrows(ArithmeticException.class, () -> {
        int i = 10 / 2;
    }, "业务逻辑居然正常运行？");
}
```

##### 超时断言

```java
@Test
@DisplayName("超时测试")
public void timeoutTest() {
    //如果测试方法时间超过1s将会异常
    Assertions.assertTimeout(Duration.ofMillis(1000), () -> Thread.sleep(500));
}
```

##### 快速失败

```java
@DisplayName("快速失败")
@Test
void testFail(){
    if(1 == 2){
        fail("测试失败");
    }
}
```

#### 前置条件（assumptions）

JUnit 5 中的前置条件（**assumptions【假设】**）类似于断言，不同之处在于**不满足的断言会使得测试方法失败**，而不满足的**前置条件只会使得测试方法的执行终止**。前置条件可以看成是测试方法执行的前提，当该前提不满足时，就没有继续执行的必要。

```java
@DisplayName("测试前置条件")
@Test
void testassumptions(){
    Assumptions.assumeTrue(false,"结果不是true");
    System.out.println("111111");

}
private final String environment = "DEV";

@Test
@DisplayName("simple")
public void simpleAssume() {
    assumeTrue(Objects.equals(this.environment, "DEV"));
    assumeFalse(() -> Objects.equals(this.environment, "PROD"));
}

@Test
@DisplayName("assume then do")
public void assumeThenDo() {
    assumingThat(
        Objects.equals(this.environment, "DEV"),
        () -> System.out.println("In DEV")
    );
}
```

assumeTrue 和 assumFalse 确保给定的条件为 true 或 false，不满足条件会使得测试执行终止。assumingThat 的参数是表示条件的布尔值和对应的 Executable 接口的实现对象。只有条件满足时，Executable 对象才会被执行；当条件不满足时，测试执行并不会终止。

#### 嵌套测试

JUnit 5 可以通过 Java 中的内部类和@Nested 注解实现嵌套测试，从而可以更好的把相关的测试方法组织在一起。在内部类中可以使用@BeforeEach 和@AfterEach 注解，而且嵌套的层次没有限制。

内部调外部 外部不能调内部

```java

@DisplayName("嵌套测试")
public class TestingAStackDemo {

    Stack<Object> stack;



    @ParameterizedTest
    @DisplayName("参数化测试")
    @ValueSource(ints = {1,2,3,4,5})
    void testParameterized(int i){
        System.out.println(i);
    }


    @ParameterizedTest
    @DisplayName("参数化测试")
    @MethodSource("stringProvider")
    void testParameterized2(String i){
        System.out.println(i);
    }


    static Stream<String> stringProvider() {
        return Stream.of("apple", "banana","atguigu");
    }

    @Test
    @DisplayName("new Stack()")
    void isInstantiatedWithNew() {
        new Stack<>();
        //嵌套测试情况下，外层的Test不能驱动内层的Before(After)Each/All之类的方法提前/之后运行
        assertNull(stack);
    }

    @Nested
    @DisplayName("when new")
    class WhenNew {

        @BeforeEach
        void createNewStack() {
            stack = new Stack<>();
        }

        @Test
        @DisplayName("is empty")
        void isEmpty() {
            assertTrue(stack.isEmpty());
        }

        @Test
        @DisplayName("throws EmptyStackException when popped")
        void throwsExceptionWhenPopped() {
            assertThrows(EmptyStackException.class, stack::pop);
        }

        @Test
        @DisplayName("throws EmptyStackException when peeked")
        void throwsExceptionWhenPeeked() {
            assertThrows(EmptyStackException.class, stack::peek);
        }

        @Nested
        @DisplayName("after pushing an element")
        class AfterPushing {

            String anElement = "an element";

            @BeforeEach
            void pushAnElement() {
                stack.push(anElement);
            }

            /**
             * 内层的Test可以驱动外层的Before(After)Each/All之类的方法提前/之后运行
             */
            @Test
            @DisplayName("it is no longer empty")
            void isNotEmpty() {
                assertFalse(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when popped and is empty")
            void returnElementWhenPopped() {
                assertEquals(anElement, stack.pop());
                assertTrue(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when peeked but remains not empty")
            void returnElementWhenPeeked() {
                assertEquals(anElement, stack.peek());
                assertFalse(stack.isEmpty());
            }
        }
    }
}
```

#### 参数化测试

参数化测试是JUnit5很重要的一个新特性，它使得用不同的参数多次运行测试成为了可能

```java
@ValueSource  // 为参数化测试指定入参来源，支持八大基础类以及String类型,Class类型
@NullSource//表示为参数化测试提供一个null的入参
@EnumSource//表示为参数化测试提供一个枚举入参
@CsvFileSource//表示读取指定CSV文件内容作为参数化测试入参
@MethodSource//表示读取指定方法的返回值作为参数化测试入参(注意方法返回需要是一个流)
```

支持外部的各类入参。如:CSV,YML,JSON 文件甚至方法的返回值也可以作为入参。只需要去实现**ArgumentsProvider**接口，任何外部文件都可以作为它的入参。

```java
@ParameterizedTest
@ValueSource(strings = {"one", "two", "three"})
@DisplayName("参数化测试1")
public void parameterizedTest1(String string) {
    System.out.println(string);
    Assertions.assertTrue(StringUtils.isNotBlank(string));
}


@ParameterizedTest
@MethodSource("method")    //指定方法名
@DisplayName("方法来源参数")
public void testWithExplicitLocalMethodSource(String name) {
    System.out.println(name);
    Assertions.assertNotNull(name);
}

static Stream<String> method() {
    return Stream.of("apple", "banana");
}
```

#### 迁移指南

在进行迁移的时候需要注意如下的变化：

- 注解在 org.junit.jupiter.api 包中，断言在 org.junit.jupiter.api.Assertions 类中，前置条件在 org.junit.jupiter.api.Assumptions 类中。
- 把@Before 和@After 替换成@BeforeEach 和@AfterEach。

- 把@BeforeClass 和@AfterClass 替换成@BeforeAll 和@AfterAll。
- 把@Ignore 替换成@Disabled。

- 把@Category 替换成@Tag。
- 把@RunWith、@Rule 和@ClassRule 替换成@ExtendWith。

### 指标监控

#### SpringBoot Actuator

每个微服务快速引用即可获得生产级别的应用监控、审计等功能。

```xml
<!--        引入监控功能-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

访问 http://localhost:8080/actuator/**

```YAML
# management 是所有actuator的配置
# management.endpoint.端点名.xxxx  对某个端点的具体配置
management:
  endpoints:
    enabled-by-default: true  #默认开启所有监控端点  true
    web:
      exposure:
        include: '*' # 以web方式暴露所有端点

  endpoint:   #对某个端点的具体配置
    health:
      show-details: always
      enabled: true

    info:
      enabled: true

    beans:
      enabled: true

    metrics:
      enabled: true
```

测试

```XML
http://localhost:8080/actuator/beans 
http://localhost:8080/actuator/configprops 
http://localhost:8080/actuator/metrics 
http://localhost:8080/actuator/metrics/jvm.gc.pause 
http://localhost:8080/actuator/endpointName/detailPath
```

#### **可视化** 

https://github.com/codecentric/spring-boot-admin

server

```xml
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-server</artifactId>
    <version>2.3.1</version>
</dependency>

@EnableAdminServer
```

client

```xml
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-client</artifactId>
    <version>2.3.1</version>
</dependency>
```

```yaml
spring:
  boot:
    admin:
      client:
        url: http://localhost:8888 #SERVER地址
        instance:
          prefer-ip: true  #使用ip注册进来
  application:
    name: boot-05-web-admin
```

#### Actuator Endpoint

##### 最常用的Endpoint 

* Health：监控状况 
* Metrics：运行时指标 
* Loggers：日志记录

##### 常使用的端点

| ID                 | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| `auditevents`      | 暴露当前应用程序的审核事件信息。需要一个`AuditEventRepository组件`。 |
| `beans`            | 显示应用程序中所有Spring Bean的完整列表。                    |
| `caches`           | 暴露可用的缓存。                                             |
| `conditions`       | 显示自动配置的所有条件信息，包括匹配或不匹配的原因。         |
| `configprops`      | 显示所有`@ConfigurationProperties`。                         |
| `env`              | 暴露Spring的属性`ConfigurableEnvironment`                    |
| `flyway`           | 显示已应用的所有Flyway数据库迁移。 需要一个或多个`Flyway`组件。 |
| `health`           | 显示应用程序运行状况信息。                                   |
| `httptrace`        | 显示HTTP跟踪信息（默认情况下，最近100个HTTP请求-响应）。需要一个`HttpTraceRepository`组件。 |
| `info`             | 显示应用程序信息。                                           |
| `integrationgraph` | 显示Spring `integrationgraph` 。需要依赖`spring-integration-core`。 |
| `loggers`          | 显示和修改应用程序中日志的配置。                             |
| `liquibase`        | 显示已应用的所有Liquibase数据库迁移。需要一个或多个`Liquibase`组件。 |
| `metrics`          | 显示当前应用程序的“指标”信息。                               |
| `mappings`         | 显示所有`@RequestMapping`路径列表。                          |
| `scheduledtasks`   | 显示应用程序中的计划任务。                                   |
| `sessions`         | 允许从Spring Session支持的会话存储中检索和删除用户会话。需要使用Spring Session的基于Servlet的Web应用程序。 |
| `shutdown`         | 使应用程序正常关闭。默认禁用。                               |
| `startup`          | 显示由`ApplicationStartup`收集的启动步骤数据。需要使用`SpringApplication`进行配置`BufferingApplicationStartup`。 |
| `threaddump`       | 执行线程转储。                                               |

如果您的应用程序是Web应用程序（Spring MVC，Spring WebFlux或Jersey），则可以使用以下附加端点：

| ID           | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| `heapdump`   | 返回`hprof`堆转储文件。                                      |
| `jolokia`    | 通过HTTP暴露JMX bean（需要引入Jolokia，不适用于WebFlux）。需要引入依赖`jolokia-core`。 |
| `logfile`    | 返回日志文件的内容（如果已设置`logging.file.name`或`logging.file.path`属性）。支持使用HTTP`Range`标头来检索部分日志文件的内容。 |
| `prometheus` | 以Prometheus服务器可以抓取的格式公开指标。需要依赖`micrometer-registry-prometheus`。 |

##### Health Endpoint

健康检查端点，我们一般用于在云平台，平台会定时的检查应用的健康状况，我们就需要Health Endpoint可以为平台返回当前应用的一系列组件健康状况的集合。

重要的几点：

- health endpoint返回的结果，应该是一系列健康检查后的一个汇总报告
- 很多的健康检查默认已经自动配置好了，比如：数据库、redis等

- 可以很容易的添加自定义的健康检查机制

##### Metrics Endpoint

提供详细的、层级的、空间指标信息，这些信息可以被pull（主动推送）或者push（被动获取）方式得到；

- 通过Metrics对接多种监控系统
- 简化核心Metrics开发

- 添加自定义Metrics或者扩展已有Metrics

##### 管理Endpoints

###### 1、开启与禁用Endpoints

- 默认所有的Endpoint除过shutdown都是开启的。

- 需要开启或者禁用某个Endpoint。配置模式为  **management.endpoint.< endpointName>.enabled = true**

  ```yaml
  management:
    endpoint:
      beans:
        enabled: true
  ```

- 或者禁用所有的Endpoint然后手动开启指定的Endpoint

  ```yaml
  management:
    endpoints:
      enabled-by-default: false
    endpoint:
      beans:
        enabled: true
      health:
        enabled: true
  ```

###### 暴露Endpoints

支持的暴露方式

- HTTP：默认只暴露**health**和**info** Endpoint
- **JMX**：默认暴露所有Endpoint

- 除过health和info，剩下的Endpoint都应该进行保护访问。如果引入SpringSecurity，则会默认配置安全访问规则

#### 定制Endpoint

##### 定制 Health 信息

```java
@Component//组件名为MyCom
public class MyComHealthIndicator extends AbstractHealthIndicator {
    /**
     * 真实的检查方法
     */
    @Override
    protected void doHealthCheck(Health.Builder builder) throws Exception {
        //mongodb。  获取连接进行测试
        Map<String,Object> map = new HashMap<>();
        // 检查完成
        if(1 == 1){
//            builder.up(); //健康
            builder.status(Status.UP);
            map.put("count",1);
            map.put("ms",100);
        }else {
//            builder.down();
            builder.status(Status.OUT_OF_SERVICE);
            map.put("err","连接超时");
            map.put("ms",3000);
        }
        builder.withDetail("code",100)
                .withDetails(map);
    }
}
//或者
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class MyHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        int errorCode = check(); // perform some specific health check
        if (errorCode != 0) {
            return Health.down().withDetail("Error Code", errorCode).build();
        }
        return Health.up().build();
    }

}

构建Health
Health build = Health.down()
                .withDetail("msg", "error service")
                .withDetail("code", "500")
                .withException(new RuntimeException())
                .build();
```

##### 定制info信息

###### 编写配置文件

```yaml
info:
  appName: boot-admin
  version: 2.0.1
  mavenProjectName: @project.artifactId@  #使用@@可以获取maven的pom文件值
  mavenProjectVersion: @project.version@
```

###### 或编写InfoContributor

```java
import java.util.Collections;
import org.springframework.boot.actuate.info.Info;
import org.springframework.boot.actuate.info.InfoContributor;
import org.springframework.stereotype.Component;
@Component
public class ExampleInfoContributor implements InfoContributor {
    @Override
    public void contribute(Info.Builder builder) {
        builder.withDetail("example",
                Collections.singletonMap("key", "value"));
    }

}
```

http://localhost:8080/actuator/info 会输出以上方式返回的所有info信息

##### 定制Metrics信息

###### SpringBoot支持自动适配的Metrics

- JVM metrics, report utilization of:

- - Various memory and buffer pools
  - Statistics related to garbage collection

- - Threads utilization
  - Number of classes loaded/unloaded

- CPU metrics
- File descriptor metrics

- Kafka consumer and producer metrics
- Log4j2 metrics: record the number of events logged to Log4j2 at each level

- Logback metrics: record the number of events logged to Logback at each level
- Uptime metrics: report a gauge for uptime and a fixed gauge representing the application’s absolute start time

- Tomcat metrics (`server.tomcat.mbeanregistry.enabled` must be set to `true` for all Tomcat metrics to be registered)
- [Spring Integration](https://docs.spring.io/spring-integration/docs/5.4.1/reference/html/system-management.html#micrometer-integration) metrics

###### 增加定制Metrics

```java
class MyService{
    Counter counter;
    public MyService(MeterRegistry meterRegistry){
         counter = meterRegistry.counter("myservice.method.running.counter");
    }
    public void hello() {
        counter.increment();
    }
}
//也可以使用下面的方式
@Bean
MeterBinder queueSize(Queue queue) {
    return (registry) -> Gauge.builder("queueSize", queue::size).register(registry);
}
```

##### 定制Endpoint

```java
@Component
@Endpoint(id = "myservice")
public class MyServiceEndPoint {
    @ReadOperation
    public Map getDockerInfo(){
        //端点的读操作  http://localhost:8080/actuator/myservice
        return Collections.singletonMap("dockerInfo","docker started.....");
    }
    @WriteOperation
    public void stopDocker(){
        System.out.println("docker stopped.....");
    }
}
```

## 高级特性

### Profile功能

多环境适配

#### application-profile功能

- 默认配置文件  application.yaml；任何时候都会加载
- 指定环境配置文件  application-{env}.yaml

- 激活指定环境

- - 配置文件激活  spring.profiles.active=dev
  - 命令行激活：java -jar xxx.jar --**spring.profiles.active=prod  --person.name=haha**

- - - **修改配置文件的任意值，命令行优先**

- 默认配置与环境配置同时生效
- 同名配置项，profile配置优先

#### @Profile条件装配功能

```java
@Configuration(proxyBeanMethods = false)
@Profile("prod")
public class ProductionConfiguration {

    // ...

}
```

#### profile分组

```properties
#设置一组
spring.profiles.group.myprod[0]=ppd  
spring.profiles.group.myprod[1]=prod
```

### 外部化配置

https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config

#### 外部配置源

常用：**Java属性文件**、**YAML文件**、**环境变量**、**命令行参数**

#### 配置文件查找位置

1. classpath 根路径
2. classpath 根路径下config目录
3. jar包当前目录
4.  jar包当前目录的config目录
5.  /config子目录的直接子目录

后覆盖前

#### 配置文件加载顺序：

1. 当前jar包内部的application.properties和application.yml
2. 当前jar包内部的application-{profile}.properties 和 application-{profile}.yml
3. 引用的外部jar包的application.properties和application.yml
4. 引用的外部jar包的application-{profile}.properties 和 application-{profile}.yml

#### 指定环境优先，外部优先，后面的可以覆盖前面的同名配置项

### 自定义starter

#### starter启动原理

- starter-pom引入 autoconfigurer 包

  starter ---> autoconfigure ---> spring-boot-starter

- autoconfigure包中配置使用 **META-INF/spring.factories** 中 **EnableAutoConfiguration 的值，使得项目启动加载指定的自动配置类**

- **编写自动配置类 xxxAutoConfiguration -> xxxxProperties**

- - **@Configuration**
  - **@Conditional**

- - **@EnableConfigurationProperties**
  - **@Bean**

- - ......

**引入starter** **--- xxxAutoConfiguration --- 容器中放入组件 ---- 绑定xxxProperties ----** **配置项**

#### 自定义starter

**atguigu-hello-spring-boot-starter（启动器）**

**atguigu-hello-spring-boot-starter-autoconfigure（自动配置包）**









































































































# Spring Cache


```java
//启动类
@EnableCaching

  

@Cacheable(value = "gathering",key="#id") //存 
@CacheEvict(value = "gathering",key="#gathering.id")//删
```
不能设置过期时间



















# Spring Data

# Spring Security

# Spring Session



# Spring Native

https://docs.spring.io/spring-native/docs/current/reference/htmlsingle/

使用[GraalVM](https://www.graalvm.org/) [本机映像](https://www.graalvm.org/reference-manual/native-image/)编译器将 Spring 应用程序编译为本机可执行文件

使用本机映像可提供关键优势，例如即时启动、即时峰值性能和减少内存消耗