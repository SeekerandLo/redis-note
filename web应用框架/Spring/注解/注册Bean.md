## 注册 Bean
1. [四大注解](#声明bean)+[包扫描](#组件扫描-@ComponentScan)

    - [组件扫描过滤规则](#过滤规则-FilterType(Enum))
    - [包扫描时加载那个函数创建组件]()

2. [@Bean](#Bean)

3. [@Import](#导入组件-@Import)

4. [FactoryBean](#FactoryBean)

5. [@Scope-控制作用域](#设置组件作用域-@Scope)

6. [@Condition-按照条件注册Bean](#按照条件注册Bean-@Condition)

7. [@Value-Bean赋值](#Bean赋值-@Value)

8. [@PropertySource-读取配置文件](#读取配置文件-@PropertySource)

## 生命周期
1. [@Bean设置initMethod和destroyMethod](#@Bean)

2. [Bean实现接口](#实现接口控制Bean)

3. [JSR250规定的注解](#@PostConstruct-&-@PreDestroy)

4. 顺序   
    - 创建对象  
    &emsp;&emsp;↓  
    - 初始化之前 [BeanPostProcessor.postProcessBeforeInitialization()](#接口-BeanPostProcessor)  
    &emsp;&emsp;↓  
    - 初始化：使用@Bean注解中的方法，或者让Bean实现接口   
    &emsp;&emsp;↓
    - 初始化之后 [BeanPostProcessor.postProcessAfterInitialization()]((#接口-BeanPostProcessor)  )  
    &emsp;&emsp;↓
    - 销毁
        - 单实例的Bean会在容器关闭的时候销毁
        - 多实例的自己管理


### 声明bean

- @Compent   
- @Service
- @Controller
- @Repository

### 


### 设置组件作用域 @Scope 
- prototype 组件设置为多例的，当设置作用域属性为多例时不会在初始化 Ioc 容器时创建，而是在使用该 Bean 时创建，每次都是新创建的

- singleton 单例的(默认值)，IoC 容器初始化时创建，之后再使用都 Bean 都是使用之前的。
    - 懒加载：**@Lazy** 容器启动时先不创建对象，当使用时才创建

### 按照条件注册Bean @Condition
- 自定义Condition
    ```java
    // 自定义
   public class MyCondition implements Condition {
        /**
        * @param context  判断上下文环境
        * @param metadata 注释信息
        */
        @Override
        public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
            // 当前 Bean 工场
            ConfigurableListableBeanFactory factory = context.getBeanFactory();
            System.out.println(factory);
            // 类
            ClassLoader classLoader = context.getClassLoader();
            System.out.println(classLoader);
            // 环境
            Environment environment = context.getEnvironment();
            // Bean 注册
            BeanDefinitionRegistry beanDefinitionRegistry = context.getRegistry();
            // 判断 return ...    
            return false;
        }
    }

    // 使用-在方法上，如加载Bean上，满足条件才会加载
    @Conditional(MyCondition.class)
    @Bean("user2")
    public User user2() {
        return new User("www", 3);
    }

    // 使用在类上，如果类满足条件，下面的Bean才会生效
    ```

#### [👉回到顶部](#注册-Bean)

### 组件扫描 @ComponentScan

- 组件注册，自动扫描组件，使用以下注解的类会被扫描添加到 IoC 容器中，**默认调用的是类中的无参构造函数**
    - @Service
    - @Component
    - @Contoller
    - @Repository 

- 过滤注解-包含
    ```java
    @ComponentScan(value = "com.liy.spring", includeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class})
    },useDefaultFilters = false)
    ```    

- 过滤注解-去除
    ```java
    @ComponentScan(value = "com.liy.spring", excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class, Service.class})})
    ```

- 包扫描时加载那个函数创建组件
    - 当 Bean 的无参构造函数和有参构造函数同时存在时，优先加载无参构造函数

#### [👉回到顶部](#注册-Bean)

### 过滤规则 FilterType(Enum)
- ANNOTATION 基于注解

- ASSIGNABLE_TYPE 基于给定的类型
    ```java
    @ComponentScan(value = "com.liy.spring", includeFilters = {
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes ={TestController.class})
    }, useDefaultFilters = false)
    ```

- ASPECTJ 使用 ASPECTJ 表达式

- REGEX 使用正则表达式匹配

- CUSTOM 自定义规则
    ```java
    // 自定义过滤器
    public class MyTypeFiler implements TypeFilter {
        /**
        * @param metadataReader        当前正在读的类的信息
        * @param metadataReaderFactory 获取到其他任何类的信息
        */
        @Override
        public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
            // 获取当前类的注解信息
            AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
            // 获取当前类的信息
            ClassMetadata classMetadata = metadataReader.getClassMetadata();
            // 获取当前类资源信息：路径
            Resource resource = metadataReader.getResource();

            String className = classMetadata.getClassName();
            // 匹配类名中包含er的类    
            return className.contains("er");
        }
    }

    // 使用过滤器
    @ComponentScan(value = "com.liy.spring", includeFilters = {
        @ComponentScan.Filter(type = FilterType.CUSTOM, classes = {
                MyTypeFiler.class
        })
    }, useDefaultFilters = false)
    ```
#### [👉回到顶部](#注册-Bean)

### 导入组件 @Import   
- 引入组件
    ```java
    @Import(Color.class)
    public class Test{

    }
    ```

- 接口：ImportSelector
    ```java
    public class MyImportSelector implements ImportSelector {
        // AnnotationMetadata 当前类的注解信息
        @Override
        public String[] selectImports(AnnotationMetadata importingClassMetadata) {
            System.out.println(importingClassMetadata);
            // 添加类 return     
            return new String[0];
        }
    }

    // 使用
    @Import(MyImportSelector.class)
    public class Test{

    }    
    ```    
- 接口：ImportBeanDefinitionRegistrar
    ```java
    public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
        /**
        * @param importingClassMetadata 当前类的注解信息
        * @param registry               Bean的注册
        */
        @Override
        public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
            // 进行判断然后注册Bean
            
        }
    }

    // 使用
    @Import(MyImportBeanDefinitionRegistrar.class)
    public class Test{

    }    
    ```
#### [👉回到顶部](#注册-Bean)

### FactoryBean
- 创建自己的FactoryBean，实现FactoryBean接口
    ```java
    public class MyFactoryBean implements FactoryBean<Color> {
        // 创建Bean时，调用此，返回一个Color对象，添加到容器中
        @Override
        public Color getObject() throws Exception {
            System.out.println("调用 ");
            return new Color();
        }

        @Override
        public Class<?> getObjectType() {
            return Color.class;
        }

        // true: 这个Bean是单实例
        // false：创建多个
        @Override
        public boolean isSingleton() {
            return true;
        }
    }

    //UserConfig2.java下  创建 
    @Bean
    public MyFactoryBean myFactoryBean() {
        return new MyFactoryBean();
    }

    // 获取
     @Test
    public void test() {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(UserConfig2.class);

        // 直接获取Bean，类型是Color，获取到是工厂Bean创建的对象
        Object factoryBean = applicationContext.getBean("myFactoryBean");

        // 加上&，获取工厂Bean，类型是MyFactoryBean
        Object factoryBean = applicationContext.getBean("&myFactoryBean");
    }
    ```

***
#### [👉回到顶部](#注册-Bean)

### @Bean
- initMethod & destroyMethod 初始化时调用 init 方法，关闭容器时调用 destroy 方法
    ```java
    // Car.java
    public class Car {
        public Car(){
            System.out.println("Car constructor");
        }

        public void init(){
            System.out.println("Car init");
        }

        public void destroy(){
            System.out.println("Car destroy");
        }
    }

    @Bean(initMethod = "init", destroyMethod = "destroy")
    public Car car() {
        return new Car();
    }
    ```
- 上述是针对单实例的 Bean 而言，如果是多实例的 Bean，在第一次使用时调用 init 方法，容器关闭不会调用 destroy 方法，自己管理 

#### [👉回到顶部](#注册-Bean)   

### 实现接口控制Bean
- 初始化创建：InitializingBean，销毁：DisposableBean
- 
    ```java
    @Component
    public class Cat implements InitializingBean, DisposableBean {
        @Override
        public void destroy() throws Exception {
            System.out.println("destroy...");
        }

        @Override
        public void afterPropertiesSet() throws Exception {
            System.out.println("init...");
        }
    }
    ```

### @PostConstruct & @PreDestroy
- ```java
  @Component
    public class Dog {
        public Dog() {
            System.out.println("创建");
        }
        // 创建赋值之后
        @PostConstruct
        public void init(){
            System.out.println("初始化");
        }
        // 销毁前
        @PreDestroy
        public void destroy(){
            System.out.println("销毁");
        }

    }
  ```

#### [👉回到顶部](#注册-Bean)

### 接口 BeanPostProcessor
- Bean 处理器，实现此接口在 Bean 初始化之前与初始化之后调用，当扫描注册到该组件时才会使用到
    ```java
    @Component
    public class MyBeanPostProcessor implements BeanPostProcessor {
        // 初始化之前
        @Override
        public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
            System.out.println("初始化之前："+beanName);
            return bean;
        }

        // 初始化之后
        @Override
        public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
            System.out.println("初始化之后："+beanName);
            return bean;
        }
    }
    ```

- 原理：调用链   
    populateBean(beanName, mbd, instanceWrapper) 给Bean赋值  
        &emsp;&emsp;&emsp;&emsp;↓  
    {  
        applyBeanPostProcessorsBeforeInitialization()  
        invokeMethods()  
        applyBeanPostProcessorsAfterInitialization()  
    }
    ***
    AbstractAutowireCapableBeanFactory
    ```java
    @Override
        public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
                throws BeansException {

            Object result = existingBean;
            for (BeanPostProcessor processor : getBeanPostProcessors()) {
                Object current = processor.postProcessBeforeInitialization(result, beanName);
                if (current == null) {
                    return result;
                }
                result = current;
            }
            return result;
        }
    ```
    ```java
    @Override
	public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
			throws BeansException {

		Object result = existingBean;
		for (BeanPostProcessor processor : getBeanPostProcessors()) {
			Object current = processor.postProcessAfterInitialization(result, beanName);
			if (current == null) {
				return result;
			}
			result = current;
		}
		return result;
	}
    ```
#### [👉回到顶部](#注册-Bean)   

***

### Bean赋值 @Value
- 支持基本数值  
    支持 SpEL 表达式：#{}  
    支持 ${}：取出配置文件中的值  

### 读取配置文件 @PropertySource
- 加载配置文件到环境变量中
- 示例，将 `pro.properties` 放在 `resource` 目录下
    ```java
    @Configuration
    @ComponentScan(basePackages = "com.liy.spring.valueconfig.bean")
    @PropertySource(value = {"classpath:/pro.properties"})
    public class ValueConfig {
        
    
    }

    // 扫描的类如下
    public class Dog{

        @Value("${dog.name}")
        private String name;

        @Value("${dog.age}")
        private String age;

        public Dog(){

        }
    }

    // 配置文件
    dog.age=10
    dog.name=qwe
    ```
- `@PropertySources` 是 PropertySources 的数组
    ```java
    @PropertySources(value = {
        @PropertySource(value = {"classpath:/pro.properties"}), 
        @PropertySource(value = {"classpath:/per.properties"})
    })
    ```
