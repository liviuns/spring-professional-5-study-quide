# Spring Professional 5 Study Guide

## Container, Dependency, and IOC

### What is dependency injection and what are the advantages?
In software engineering, dependency injection is a technique whereby one object (or static method) supplies the dependencies of another object. A dependency is an object that can be used (a service).

### What is an interface and what are the advantages of making use of them in Java? 
A particular advantage of using interface in Java is that it allows multiple inheritance. The full power of Interface is utilized when dependency injection techniques is used to inject required implementation on run time.

### Why are they recommended for Spring beans?
When Spring beans are implementing interfaces, it is very easy to change the application functionality. You can simply exchange entire beans by changing the Spring configuration, but you don't have to make any changes to the code.

### What is meant by "application-context"? 
The Application Context is Spring's advanced container. Similar to BeanFactory, it can load bean definitions, wire beans together, and dispense beans upon request. Additionally, it adds more enterprise-specific functionality such as the ability to resolve textual messages from a properties file and the ability to publish application events to interested event listeners. This container is defined by org.springframework.context.ApplicationContext interface.

The ApplicationContext includes all functionality of the BeanFactory, It is generally recommended over BeanFactory. BeanFactory can still be used for lightweight applications like mobile devices or applet-based applications.

### How are you going to create a new instance of an ApplicationContext? 
```
// Loading XML configuration from the class path
ApplicationContext context = 
new ClassPathXmlApplicationContext("bean-definitions.xml");
```
```
// Loading XML configuration from the file
ApplicationContext context = 
new FileSystemXmlApplicationContext("path/to/bean-definitions.xml");
```
```
// Loading Java based configuration
ApplicationContext context = 
new AnnotationConfigApplicationContext(BeanDefinitions.class);
```
```
// Loading Java based configuration of the web application
ApplicationContext context = 
new AnnotationConfigWebApplicationContext();
```
### Can you describe the lifecycle of a Spring Bean in an ApplicationContext? 
-Bean definition is loaded from the configuration
-Bean definition is processed by the BeanFactoryPostProcessors
-Bean instance is created. Required dependencies are provided and set by the container
-All optional dependencies are provided and set by the container
-Bean is processed by the BeanPostProcessor#postProcessBeforeInitialization()
-Initialization callbacks are executed (in order @PostConstruct annotation, InitializingBean#afterPropertiesSet() method and custom configured init() method).
-Bean is processed by the BeanPostProcessor#postProcessAfterInitialization()
-The container and all the beans are now ready to be used.
-When the application is stopped, destroy callbacks are executed (in order @PreDestroy annotation, DisposableBean#destroy() method and custom configured destroy() method). Destroy callbacks are not called for the Prototype scoped beans!

### How are you going to create an ApplicationContext in an integration test? 
```
@RunWith(SpringRunner.class)
@ContextConfiguration(classes = {AppConfig.class, ITConfig.class}) 
public class myIT {}
```
You can also use @WebContextConfiguration

### What is the preferred way to close an application context? Does Spring Boot do this for you?
For non-web application, you should register a shutdown hook to ensure that ApplicationContext will be closed gracefully and it calls the relevant destroy methods on singleton beans (remember, that destroy callbacks are not called on prototype-scoped beans). You can register shutdown hook via ConfigurableApplicationContext. If you are using Spring Boot, then it will register this hook automatically for you.
example: 
```
ConfigurableApplicationContext ctx = new
  SpringApplicationBuilder(Application.class).web(WebApplicationType.NONE).run();
System.out.println("Spring Boot application started");
ctx.getBean(TerminateBean.class);
ctx.close();
```

### Can you describe: 
### Dependency injection using Java configuration? 
The central artifact in Spring's new Java-configuration support is the @Configuration-annotated class. These classes consist principally of @Bean-annotated methods that define instantiation, configuration, and initialization logic for objects that are managed by the Spring IoC container.
Annotating a class with the @Configuration indicates that the class can be used by the Spring IoC container as a source of bean definitions. The simplest possible @Configuration class would read as follows:
```
@Configuration
public class AppConfig {}
```
An application may use one @Configuration-annotated class, or many. @Configuration is meta-annotated as a @Component. Therefore, @Configuration-annotated classes are candidates for component-scanning and can also take advantage of @Autowired annotations at the field and method levels, but not at the constructor level. @Configuration-annotated classes must also have a default constructor. You can wire externalized values into @Configuration-annotated classes with the @Value annotation.

### Dependency injection using annotations (@Autowired)? 
In Spring, you can use @Autowired annotation to auto wire bean on the setter method, constructor or a field. Moreover, it can autowired property in a particular bean. The @Autowired annotation is auto wire the bean by matching data type.

### Component scanning, Stereotypes?
```
@Component, @Controller, @Service, and @Repository
```
Spring can automatically detect your stereotype classes and instantiate beans defined inside those classes. To allow autodetection, it is needed to add the @ComponentScan annotation on your @Configuration class.

### Scopes for Spring beans? What is the default scope?
The default scope for a Spring bean is singleton

### Are beans lazily or eagerly instantiated by default? How do you alter this behavior?
Beans are eagerly instantiated by default. You can change this behavour by adding the @Lazy annotation to a bean
```
@Bean
@Lazy
public MyBean myBean(){}
```

### What is a property source? How would you use @PropertySource? 
Spring @PropertySource annotation is used to provide properties file to Spring Environment. Default properties are stored in application.properties.
```
@Configuration
	@PropertySources({
		@PropertySource("classpath:config.properties"),
		@PropertySource("classpath:db.properties")
	})
	public class AppConfig {
		//...
	}
```

### What is a BeanFactoryPostProcessor and what is it used for? When is it invoked? 
The BeanFactoryPostProcessor is a special kind of object registered in ApplicationContext . Its main purpose is to alter BeanFactory before the beans are instantiated. That means, that BeanFactoryPostProcessor operates on raw bean definitions, not on the bean instances.
After loading the bean definitions from all styles of configurations, BeanFactoryPostProcessor comes into the picture to modify the definition of some beans, and then the container instantiates the beans. Finally, BeanPostProcessor works on the beans, and it can modify and change the bean object. This is the initialization phase.

### Why would you define a static @Bean method?
By marking a bean as static, it can be invoked without causing instantiation of its declaring @Configuration class, thus avoiding lifecycle conflicts. Note however that static @Bean methods will not be enhanced for scoping and AOP semantics. This works out in BFPP cases, as they are not typically referenced by other @Bean methods. As a reminder, a WARN-level log message will be issued for any non-static @Bean methods having a return type assignable to BeanFactoryPostProcessor.

### What is a ProperySourcesPlaceholderConfigurer used for?
The PropertySourcesPlaceholderConfigurer is Springs internal implementation of the BeanFactoryPostProcessor . Its main purpose is to resolve ${...} placeholders within bean definition property values and @Value annotations with appropriate values loaded from the property sources.

### What is a BeanPostProcessor and how is it different to a BeanFactoryPostProcessor? What do they do? When are they called?
The BeanFactoryPostProcessor executes before bean Object instantiation (ie at the time Applicationcontext container is initialized)
BeanPostprocessor is executed after the bean object is created, as it can be executed before init() and after init().

### What is an initialization method and how is it declared on a Spring bean?
In Spring, you can use init-method and destroy-method as attribute in bean configuration file for bean to perform certain actions upon initialization and destruction.
Implementing the org.springframework.beans.factory.InitializingBean interface allows a bean to perform initialization work after all necessary properties on the bean have been set by the container. The InitializingBean interface specifies exactly one method:

void afterPropertiesSet() throws Exception;
Generally, the use of the InitializingBean interface can be avoided and is actually discouraged since it unnecessarily couples the code to Spring. As an alternative, bean definitions provide support for a generic initialization method to be specified. In the case of XML-based configuration metadata, this is done using the 'init-method' attribute. For example, the following definition:
```
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
public class ExampleBean {   
    public void init() {
        // do some initialization work
    }
}
```
...is exactly the same as...
```
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
public class AnotherExampleBean implements InitializingBean {   
    public void afterPropertiesSet() {
        // do some initialization work
    }
}
```

### What is a destroy method, how is it declared and when is it called?
Implementing the org.springframework.beans.factory.DisposableBean interface allows a bean to get a callback when the container containing it is destroyed. The DisposableBean interface specifies a single method:
```void destroy() throws Exception;```

### Consider how you enable JSR-250 annotations like @PostConstruct and @PreDestroy? When/how will they get called?
The @PostConstruct annotation is used on a method that needs to be executed after dependency injection is done to perform any initialization.
The @PreDestroy annotation is used on methods as a callback notification to signal that the instance is in the process of being removed by the container.

### How else can you define an initialization or destruction method for a Spring bean?
Through xml
```
<bean id="initAndDestroyServiceConfiguration" name="initAndDestroyServiceConfiguration" 
      init-method="postInitMethod" destroy-method="preDestroyMethod" 
      class="com.ronan.example.spring.bean.initdestroy.InitAndDestroyServiceConfiguration"> 
</bean>
```

### What does component-scanning do?
Spring is a dependency injection framework. It is all about beans and wiring in dependencies.
The first step of defining Spring Beans is by adding the right annotation - @Component or @Service or @Repository.
However, Spring does not know about the bean unless it knows where to search for it.
This part of “telling Spring where to search” is called a Component Scan.
You define the packages that have to be scanned.

Once you define a Component Scan for a package, Spring would search the package and all its sub packages for components/beans.
@SpringBootApplication defines an automatic component scan of the package the annotaion resides.

### What is the behavior of the annotation @Autowired with regards to field injection, constructor injection and method injection?
This annotation is applied to fields, setter methods, and constructors. The @Autowired annotation injects object dependency implicitly. When you use @Autowired on fields and pass the values for the fields using the property name, Spring will automatically assign the fields with the passed values. You can even use @Autowired  on private properties.

When you use @Autowired on setter methods, Spring tries to perform it by Type autowiring on the method. You are instructing Spring that it should initiate this property using a setter method where you can add your custom code.

### What do you have to do, if you would like to inject something into a private field? How does this impact testing?
There are two basic cases of injecting a value or reference into a private field; either the source- code containing the private field to be injected can be modified or it cannot be modified. The latter case typically occurs with third-party libraries and generated code. Some of the following is only applicable in the case where the source-code can be modified.

### How does the @Qualifier annotation complement the use of @Autowired?
The @Qualifier annotation adds additional control of the selection of the bean to inject if there are multiple beans of the same type in the Spring application context.
The @Qualifier annotation can be used at three different locations:
• At injection points.
• At bean definitions.
• At annotation definitions.
This creates a custom qualifier annotation.

### What is a proxy object and what are the two different types of proxies Spring can create? 
The Spring framework is able to create two types of proxy objects:
• JDK Dynamic Proxy
• CGLIB Proxy
A proxy object is an object that have the same methods, at least the public methods, as the object it proxies. The purpose of this is to make the proxy indistinguishable from the object it proxies. The proxy object contains a reference to the object it proxies. When a reference to the original, proxied, object is requested, a reference to the proxy object is supplied. 

### What are the limitations of these proxies (per type)? 
### What is the power of a proxy object and where are the disadvantages?
Can add behavior to existing beans i.e. Transaction management, logging, security.
Makes it possible to separate concerns such as logging, security etc from business logic.

### What does the @Bean annotation do?
@Bean annotation indicates that the annotated method produces a bean to be managed by the Spring container.

### What is the default bean id if you only use @Bean? How can you override this?
```
@Bean
public Service service(){
}
```
the above bean name is service. To change it to myService we could override with the following:
```
@Bean(name = "myService")
public Service service(){
}
```

### Why are you not allowed to annotate a final class with @Configuration 
The Spring container will create a subclass of each class annotated with @Configuration when creating an application context using CGLIB. Final classes cannot be subclassed.

### How do @Configuration annotated classes support singleton beans? 
### Why can’t @Bean methods be final either? 
Spring creates dynamic proxies for classes annotated with @Configuration classes. Spring uses CGLIB to extend your class to create proxy. Hence, configuration classes or beans cannot be final.

### How do you configure profiles? What are possible use cases where they might be useful?
Profiles are a core feature of the framework – allowing us to map our beans to different profiles – for example, dev, test, prod. We can then activate different profiles in different environments to bootstrap just the beans we need.

### Can you use @Bean together with @Profile?
Short answer yes

### Can you use @Component together with @Profile?
Not sure //TODO
### How many profiles can you have?
You can have multipe profile troughout your code base

### How do you inject scalar/literal values into Spring beans?
Let’s say we have the following key-value pairs in a file app.properties:
```
app.string.property=hello
app.integer.property=987
app.floating.point.property=3.14159
app.boolean.property=false
To get access to these properties, we declare an application configuration in Java:
```
```
@Configuration
@ComponentScan
@PropertySource("app.properties")
public class AppConfig {}
```

### What is @Value used for?
This annotation can be used for injecting values into fields in Spring-managed beans and it can be applied at 
```
field 
constructor
method
```

### What is Spring Expression Language (SpEL for short)?
The Spring Expression Language (SpEL) is a powerful expression language that supports querying and manipulating an object graph at runtime. It can be used with XML or annotation-based Spring configurations.

There are several operators available in the language:
```
Type	Operators
Arithmetic	+, -, *, /, %, ^, div, mod
Relational	<, >, ==, !=, <=, >=, lt, gt, eq, ne, le, ge
Logical	and, or, not, &&, ||, !
Conditional	?:
Regex	matches
```

### What is the Environment abstraction in Spring? 
### Where can properties in the environment come from – there are many sources for properties – check the documentation if not sure. Spring Boot adds even more. 
### What can you reference using SpEL? 
### What is the difference between $ and # in @Value expressions? 
## Aspect oriented programming 
### What is the concept of AOP? Which problem does it solve? What is a cross cutting concern? 
### Name three typical cross cutting concerns. 
### What two problems arise if you don't solve a cross cutting concern via AOP? 
### What is a pointcut, a join point, an advice, an aspect, weaving? 
### How does Spring solve (implement) a cross cutting concern? 
### Which are the limitations of the two proxy###types? 
### What visibility must Spring bean methods have to be proxied using Spring AOP? 
### How many advice types does Spring support. Can you name each one? 
### What are they used for? 
### Which two advices can you use if you would like to try and catch exceptions? 
### What do you have to do to enable the detection of the @Aspect annotation? 
### What does @EnableAspectJAutoProxy do?
Enables support for handling components marked with AspectJ's @Aspect annotation

### If shown pointcut expressions, would you understand them? March 2019 © Copyright 2019 Pivotal Software, Inc. All rights reserved 7 
### For example, in the course we matched getter methods on Spring Beans, what would be the correct pointcut expression to match both getter and setter methods? 
### What is the JoinPoint argument used for? 
### What is a ProceedingJoinPoint? When is it used?

## Data Management: JDBC, Transactions 
### What is the difference between checked and unchecked exceptions?
From Spring > 'Checked exceptions' represent errors outside the control of the program. If a program throws an 'Unchecked exception', it reflects some error inside the program logic.

### Why does Spring prefer unchecked exceptions?
Unchecked exceptions gives a developer the freedom to decide where to implement error handling.

### What is the data access exception hierarchy?
This exception hierarchy aims to let user code find and handle the kind of error encountered without knowing the details of the particular data access API in use (e.g. JDBC). Thus it is possible to react to an optimistic locking failure without knowing that JDBC is being used.

### How do you configure a DataSource in Spring? Which bean is very useful for development/test databases?
Spring Boot uses an opinionated algorithm to scan for and configure a DataSource. This allows us to easily get a fully-configured DataSource implementation by default.

In addition, Spring Boot automatically configures a lightning-fast connection pool — either HikariCP, Apache Tomcat, or Commons DBCP, in that order, depending on which are on the classpath.

### What is the Template design pattern and what is the JDBC template?
The Template Design Pattern is a behavior pattern and, as the name suggests, it provides a template or a structure of an algorithm which is used by users. A user provides its own implementation without changing the algorithm’s structure.

The central class of the Spring JDBC abstraction framework is the JdbcTemplate class that includes the most common logic in using the JDBC API to access data, such as handling the creation of connection, statement creation, statement execution, and release of resource.

### What is a callback? What are the three JdbcTemplate callback interfaces that can be used with queries? What is each used for? (You would not have to remember the interface names in the exam, but you should kn ow what they do if you see them in a code sample). 
### Can you execute a plain SQL statement with the JDBC template?
Short answer yes
```
statement.executeQuery("SELECT * FROM student")
```

### When does the JDBC template acquire (and release) a connection, for every method called or once per template? Why?
JDBC template will acquire a connect before and after evert method call and release after every method call.

### How does the JdbcTemplate support generic queries? How does it return objects and lists/maps of objects?

### What is a transaction? What is the difference between a local and a global transaction? 
Transaction is an indivisible unit of work. 

Transactions are described in terms of ACID properties:Atomic, Consistent, Isolated, Durable.

local transaction is a simple transaction that is about one single database; whilst a global one is application server managed and spreads across many components/ technologies.

### Is a transaction a cross cutting concern? How is it implemented by Spring?
Yes and its implemented using Spring AOP

### How are you going to define a transaction in Spring?
You can define transactions in 2 ways: Declarative(xml) way or Programmatic(annotated) way

### What does @Transactional do? What is the PlatformTransactionManager?
When using proxies, you should apply the @Transactional annotation only to methods with public visibility. If you do annotate protected, private or package-visible methods with the @Transactional annotation, no error is raised, but the annotated method does not exhibit the configured transactional settings. Consider the use of AspectJ (see below) if you need to annotate non-public methods.

Spring recommends that you only annotate concrete classes (and methods of concrete classes) with the @Transactional annotation, as opposed to annotating interfaces. You certainly can place the @Transactional annotation on an interface (or an interface method), but this works only as you would expect it to if you are using interface-based proxies. The fact that Java annotations are not inherited from interfaces means that if you are using class-based proxies ( proxy-target-class="true") or the weaving-based aspect ( mode="aspectj"), then the transaction settings are not recognized by the proxying and weaving infrastructure, and the object will not be wrapped in a transactional proxy, which would be decidedly bad.

PlatformTransactionManager is the central interface in Spring's transaction infrastructure. Applications can use this directly, but it is not primarily meant as API: Typically, applications will work with either TransactionTemplate or declarative transaction demarcation through AOP.
For implementors, it is recommended to derive from the provided AbstractPlatformTransactionManager class, which pre-implements the defined propagation behavior and takes care of transaction synchronization handling. Subclasses have to implement template methods for specific states of the underlying transaction, for example: begin, suspend, resume, commit.

### Is the JDBC template able to participate in an existing transaction? 
### What is a transaction isolation level? How many do we have and how are they ordered? 
### What is @EnableTransactionManagement for? 
Enables Spring's annotation-driven transaction management capability, similar to the support found in Spring's <tx:*> XML namespace. To be used on @Configuration classes as follows
```
@Configuration
@EnableTransactionManagement
public class AppConfig {
}
 ```
### What does transaction propagation mean? 
Defines how transactions relate to each other. Common options:
 -Required: Code will always run in a transaction. Creates a new transaction or reuses one if available.
 -Requires_new: Code will always run in a new transaction. Suspends the current transaction if one exists.
 
### What happens if one @Transactional annotated method is calling another @Transactional annotated method on the same object instance? 
If you call a method with a @Transactional annotation from a method with @Transactional within the same instance, then the called methods transactional behavior will not have any impact on the transaction. But if you call a method with a transaction definition from another method with a transaction definition, and they are in different instances, then the code in the called method will follow the transaction definitions given in the called method.

### Where can the @Transactional annotation be used? What is a typical usage if you put it at class level? 
### What does declarative transaction management mean?
Declarative transaction management approach allows you to manage the transaction with the help of configuration instead of hard coding in your source code. This means that you can separate transaction management from the business code. You only use annotations or XML-based configuration to manage the transactions.

### What is the default rollback policy? How can you override it? 
### What is the default rollback policy in a JUnit test, when you use the @RunWith(SpringJUnit4ClassRunner.class) in JUnit 4 or @ExtendWith(SpringExtension.class) in JUnit 5, and annotate your @Test annotated method with @Transactional? 
### Why is the term "unit of work" so important and why does JDBC AutoCommit violate this pattern? 
### What do you need to do in Spring if you would like to work with JPA? 
### Are you able to participate in a given transaction in Spring while working with JPA? 
### Which PlatformTransactionManager(s) can you use with JPA?
### What do you have to configure to use JPA with Spring? How does Spring Boot make this easier? 

## Spring Data JPA 
### What is a Repository interface?
### How do you define a Repository interface? Why is it an interface not a class? 
The central interface in the Spring Data repository abstraction is Repository. It takes the domain class to manage as well as the ID type of the domain class as type arguments. This interface acts primarily as a marker interface to capture the types to work with and to help you to discover interfaces that extend this one. The CrudRepository provides sophisticated CRUD functionality for the entity class that is being managed.
```
public interface CrudRepository<T, ID> extends Repository<T, ID> {

  <S extends T> S save(S entity);      

  Optional<T> findById(ID primaryKey); 

  Iterable<T> findAll();               

  long count();                        

  void delete(T entity);               

  boolean existsById(ID primaryKey);   

  // … more functionality omitted.
}
```

### What is the naming convention for finder methods in a Repository interface?
findByX

### How are Spring Data repositories implemented by Spring at runtime?
First of all, there's no code generation going on, which means: no CGLib, no byte-code generation at all. The fundamental approach is that a JDK proxy instance is created programmatically using Spring's ProxyFactory API to back the interface and a MethodInterceptor intercepts all calls to the instance and routes the method into the appropriate places

### What is @Query used for?
The @Query annotation declares finder queries directly on repository methods. While similar @NamedQuery is used on domain classes, Spring Data JPA @Query annotation is used on Repository interface. This frees the domain classes from persistence specific information, which is a good thing.

## Spring MVC and the Web Layer 
### MVC is an abbreviation for a design pattern. What does it stand for and what is the idea behind it?
MVC is an abbreviation for Model-View-Controller design pattern. This pattern is based upon the separation-of-concerns design principle that promotes handling different functionality at different layers and loose coupling between layers.

### What is the DispatcherServlet and what is it used for? 
The DispatcherServlet is a front controller like it provides a single entry point for a client request to Spring MVC web application and forwards request to Spring MVC controllers for processing.

### What is a web application context? What extra scopes does it offer? 
WebApplicationContext is a Spring application context for web applications.

WebApplicationContext has all the properties of a regular Spring application contex, given that the WebApplicationContext interface extends the ApplicationContext interface

WebApplicationContext can access the Servlet Context.

### What is the @Controller annotation used for? 
### How is an incoming request mapped to a controller and mapped to a method? 
### What is the difference between @RequestMapping and @GetMapping?
@GetMapping is a composed annotation that acts as a shortcut for @RequestMapping(method = RequestMethod.GET).
@GetMapping is the newer annotaion. It supports consumes

### What is @RequestParam used for?
@RequestParam is a Spring annotation used to bind a web request parameter to a method parameter. It has the following optional elements: defaultValue - used as a fallback when the request parameter is not provided or has an empty value.

### What are the differences between @RequestParam and @PathVariable?
Both the the annotations have the significant purpose and use respectively. The key difference between @RequestParam and @PathVariable is that @RequestParam used for accessing the values of the query parameters where as @PathVariable used for accessing the values from the URI template.

### What are some of the parameter types for a controller method?
* ServletRequest or HttpServletRequest.
* HttpSession. 
* NativeWebRequest
* Locale 
* InputStream 
* OutputStream
* Principal
* @PathVariable
* @RequestParam 
* @RequestHeader 
* @RequestBody
* @RequestPart 
* HttpEntity<?>
* Model/ModelMap
* RedirectAttributes
* BindingResult 
* SessionStatus 
* UriComponentsBuilder

### What are some of the valid return types of a controller method? 
ModelAndView (Class)
Model (Interface)
Map.
String.
void.
HttpHeaders.

## Security 
### What are authentication and authorization? Which must come first? 
### Is security a cross cutting concern? How is it implemented internally? 
### What is the delegating filter proxy? 
### What is the security filter chain? 
### What is a security context? 
### What does the ** pattern in an antMatcher or mvcMatcher do? 
### Why is the usage of mvcMatcher recommended over antMatcher? 
### Does Spring Security support password hashing? What is salting? 
### Why do you need method security? What type of object is typically secured at the method level (think of its purpose not its Java type). 
### What do @PreAuthorized and @RolesAllowed do? What is the difference between them?
@Secured and @RolesAllowed are the same the only difference is @RolesAllowed is a standard annotation (i.e. not only spring security) whereas @Secured is spring security only.
@PreAuthorize is different in a way that it is more powerful then the other 2. It allows for SpEL expression for a more fine-grained control. Which to use well the simplest thing that could possible work, if you don’t need expression etc. go with the standard annotations to limit the dependency on spring classes. 

### How are these annotations implemented? 
### In which security annotation are you allowed to use SpEL?
Both @PreAuthorize and @PostAuthorize annotations provide expression-based access control. Hence, predicates can be written using SpEL (Spring Expression Language)

## REST 
### What does REST stand for? 
Representational State Transfer
### What is a resource? 
Java supports javax.annotation.Resource annotation using JSR-250 that is applied on component class and fields or methods of component class. When @Resource is applied on fields or methods, the container injects the requested resource. Spring supports @Resource annotation. It behaves same as @Autowired annotation of spring. @Autowired annotation automatically injects the required bean. @Resource has name attribute. It can be used in two ways.

example
```
@Resource(name="person")
private Person person; 
```
Note: name attribute is not required

### What does CRUD mean? 
CREATE,READ,UPDATE,DELETE

### Is REST secure? What can you do to secure it? 
### Is REST scalable and/or interoperable?
Yes, REST is Scalable and interoperable. It doesn't mandate a specific choice of technology either at client or server end. You can use Java, C++, Python or JavaScript to create RESTful Web Services and Consume them at the client end.

### Which HTTP methods does REST use?
The primary or most-commonly-used HTTP verbs (or methods, as they are properly called) are POST, GET, PUT, PATCH, and DELETE. These correspond to create, read, update, and delete (or CRUD) operations, respectively.

### What is an HttpMessageConverter?
Internally Spring MVC uses a component called a HttpMessageConverter to convert the Http request to an object representation and back. A set of default converters are automatically registered which supports a whole range of different resource representation formats - json, xml for instance

### Is REST normally stateless?
REST is stateless and doesn't maintain any states between the requests.

### What does @RequestMapping do? 
@RequestMapping can be applied to the controller class as well as methods. 
You use the @RequestMapping annotation to map URLs such as /appointments onto an entire class or a particular handler method.

### Is @Controller a stereotype? Is @RestController a stereotype?
@RestController(No) is a convenience annotation that is itself annotated with @Controller(Yes) 

### What is a stereotype annotation? What does that mean?
A stereotype allows a framework developer to identify such a role and declare some common metadata for beans with that role in a central place.

### What is the difference between @Controller and @RestController? 
The @RestController annotation in Spring MVC is nothing but a combination of @Controller and @ResponseBody

### When do you need @ResponseBody?
@ResponseBody means that the controller's return value should be used as the body of the response

### What are the HTTP status return codes for a successful GET, POST, PUT or DELETE operation?
For a PUT request: HTTP 200 or HTTP 204 should imply "resource updated successfully". For a DELETE request: HTTP 200 or HTTP 204 should imply "resource deleted successfully". HTTP 202 can also be returned which would imply that the instruction was accepted by the server and the "resource was marked for deletion"

### When do you need @ResponseStatus?
We can use @ResponseStatus to mark a method or an exception class with a status code and reason that should be returned.

### Where do you need @ResponseBody? What about @RequestBody? Try not to get these muddled up!
@RequestBody : Annotation indicating a method parameter should be bound to the body of the HTTP request.
@ResponseBody annotation can be put on a method and indicates that the return type should be written straight to the HTTP response body (and not placed in a Model, or interpreted as a view name).

### If you saw example Controller code, would you understand what it is doing? Could you tell if it was annotated correctly? 
### Do you need Spring MVC in your classpath? 
The short answer is: yes — you need Spring MVC in your Java application's classpath to develop RESTful web services using the Spring framework.

### What Spring Boot starter would you use for a Spring REST application? 
### What are the advantages of the RestTemplate? 
### If you saw an example using RestTemplate would you understand what it is doing?
The RestTemplate is the basic Spring class for simultaneous client-side HTTP access. It simplifies the interaction with HTTP servers and enforces RESTful systems. It is very related to the JdbcTemplate, JmsTemplate and the various other templates. 

Why should we use RestTemplate:-
Basically, RestTemplate is used to make HTTP Rest Calls (REST Client).
If we want to make an HTTP Call, we need to create an HttpClient, pass request and form parameters, setup accept headers and perform unmarshalling of response, all by yourself, Spring Rest Templates tries to take the pain away by abstracting all these details from you.
It is thread-safe, once created you can use it as a callbacks to customize its works.

## Testing 
### Do you use Spring in a unit test?
A unit test tests one unit of functionality, a method in a class, a class or an entire module. The unit tested is tested in isolation outside of the environment for which it is intended. This means, among other things, that the Spring framework is not used to perform dependency injection in unit tests. Instead any dependencies are commonly replaced by mocks or stubs, which are created programmatically in the test-class, and set on the instance-under-test using setter-methods.
Thus Spring is not commonly used in unit tests.

### What type of tests typically use Spring?
Integration tests

### How can you create a shared application context in a JUnit integration test?
If the question is regarding how to define a Spring application context in one single place which can be used in multiple JUnit integration tests then this can be accomplished using the @ContextConfiguration annotation. This annotation is used at class level in integration test classes and allows for specifying a number of @Configuration classes, or resource locations, used to load a Spring application context. In order to define a Spring application context in one single place that is later used by multiple integration tests a class that is a common superclass to all the integration tests classes can be defined and annotated with the @ContextConfiguration annotation.
Subclasses will, as per default, inherit the @Configuration classes, resource locations and context initializers. Subclasses may append to the list of @Configuration classes or resource locations used to load a Spring application context for integration tests by specifying one or more @Configuration classes, resource locations or context initializers.

### When and where do you use @Transactional in testing? 
The @Transactional annotation can be used in a test that alter some transactional resource, for example a database, that is to be restored to the state it had prior to the test being run. You can also use @Transactional at test method level

### How are mock frameworks such as Mockito or EasyMock used? 
Mockito and EasyMock allows for dynamic creation of mock objects that can be used to mock collaborators of class(es) under test that are external to the system or trusted.
A mock object is similar to a stub, in that it produces predetermined results when methods on the object are invoked.

### How is @ContextConfiguration used? 
### How does Spring Boot simplify writing tests? 
### What does @SpringBootTest do? How does it interact with @SpringBootApplication and @SpringBootConfiguration?
@SpringBootConfiguration is an alternative to the @Configuration annotation. The main difference is that @SpringBootConfiguration allows configuration to be automatically located. This can be especially useful for unit or integration tests.

The recommendation is to only have one @SpringBootConfiguration or @SpringBootApplication for your application. Most applications will simply use @SpringBootApplication.

With the @SpringBootTest annotation, Spring Boot provides a convenient way to start up an application context to be used in a test.

@SpringBootTest by default starts searching in the current package of the test class and then searches upwards through the package structure, looking for a class annotated with @SpringBootConfiguration from which it then reads the configuration to create an application context. This class is usually our main application class since the @SpringBootApplication annotation includes the @SpringBootConfiguration annotation. It then creates an application context very similar to the one that would be started in a production environment.

## Spring Boot Intro 
### What is Spring Boot? 
Spring Boot is a project gathering a number of modules under a common umbrella. Some of the more central modules are: 
### What are the advantages of using Spring Boot? 
Does not generate code. Provides a set of managed dependencies that have been verified to work together. Provides a set of managed Maven plug-ins configured to produce certain artifacts.

### Why is it “opinionated”?
By design, the software is limiting and encourages the designer into doing things their way. In other words, Spring thinks you should develop an application the way Spring framework thinks best but dosnt force you to do so.

### What things affect what Spring Boot sets up? 
### What is a Spring Boot starter POM? Why is it useful? 
Starters are a set of convenient dependency descriptors that you can include in your application
### Spring Boot supports both properties and YML files. Would you recognize and understand them if you saw them? 
### Can you control logging with Spring Boot? How? 
### Where does Spring Boot look for property file by default?
By default, Spring searches for the property files at the root. If we have to specify a particular folder we can provide the location via searchPaths.

### How do you define profile specific property files? 
### How do you access the properties defined in the property files? 
### What properties do you have to define in order to configure external MySQL? 
### How do you configure default schema and initial data? 
### What is a fat far? How is it different from the original jar?
Basically, a fat jar (also known as uber-jar) is a self-sufficient archive which contains both classes and dependencies needed to run an application.

By default, Boot creates a jar file – but if we change the packaging property in pom.xml to war, Maven will instead naturally build a war.

### What is the difference between an embedded container and a WAR?
Most server-side Java applications (e.g. web or service-oriented) are intended to run within a container. The traditional way to package these apps for distribution is to bundle them as a WAR file. ... Rather than being deployed to a container, an embedded container is deployed within the application itself!

### What embedded containers does Spring Boot support? 
Tomcat (default), Jetty and Undertow

## Spring Boot Auto Configuration 
### How does Spring Boot know what to configure?
Simply put, the Spring Boot autoconfiguration represents a way to automatically configure a Spring application based on the dependencies that are present on the classpath. This can make development faster and easier by eliminating the need for defining certain beans that are included in the auto-configuration classes.

### What does @EnableAutoConfiguration do?
Enable auto-configuration of the Spring Application Context, attempting to guess and configure beans that you are likely to need. Auto-configuration classes are usually applied based on your classpath and what beans you have defined. For example, if you have tomcat-embedded.jar on your classpath you are likely to want a TomcatServletWebServerFactory (unless you have defined your own ServletWebServerFactory bean).
When using SpringBootApplication, the auto-configuration of the context is automatically enabled and adding this annotation has therefore no additional effect.

Auto-configuration tries to be as intelligent as possible and will back-away as you define more of your own configuration. You can always manually exclude() any configuration that you never want to apply (use excludeName() if you don't have access to them). You can also exclude them via the spring.autoconfigure.exclude property. Auto-configuration is always applied after user-defined beans have been registered.

The package of the class that is annotated with @EnableAutoConfiguration, usually via @SpringBootApplication, has specific significance and is often used as a 'default'. For example, it will be used when scanning for @Entity classes. It is generally recommended that you place @EnableAutoConfiguration (if you're not using @SpringBootApplication) in a root package so that all sub-packages and classes can be searched.

Auto-configuration classes are regular Spring Configuration beans. They are located using the SpringFactoriesLoader mechanism (keyed against this class). Generally auto-configuration beans are @Conditional beans (most often using @ConditionalOnClass and @ConditionalOnMissingBean annotations).

### What does @SpringBootApplication do? 
The @SpringBootApplication is a convenience-annotation that can be applied to Spring Java configuration classes. The @SpringBootApplication is equivalent to the three annotations @Configuration, @EnableAutoConfiguration and @ComponentScan.

### Does Spring Boot do component scanning? Where does it look by default?
With Spring, we use the @ComponentScan annotation along with @Configuration annotation to specify the packages that we want to be scanned. @ComponentScan without arguments tells Spring to scan the current package and all of its sub-packages.

### How are DataSource and JdbcTemplate auto-configured? 
### What is spring.factories file for? 
### How do you customize Spring auto configuration? 
To create a custom auto-configuration, we need to create a class annotated as @Configuration and register it.
Add it to > resources/META-INF/spring.factories:

### What are the examples of @Conditional annotations? How are they used? 

## Spring Boot Actuator 
### What value does Spring Boot Actuator provide? 
Actuator brings production-ready features to our application.
Monitoring our app, gathering metrics, understanding traffic or the state of our database becomes trivial with this dependency. The main benefit of this library is that we can get production grade tools without having to actually implement these features ourselves.

### What are the two protocols you can use to access actuator endpoints? 
HTTP, JMX
### What are the actuator endpoints that are provided out of the box?
```
autoconfig,
beans,
configprops,
dump,
env,
health,
info,
metrics,
mappings,
shutdown,
trace,
logger
```

### What is info endpoint for? How do you supply data?
The info endpoint (http://localhost:8080/actuator/info) displays general information about your application obtained from build files like META-INF/build-info.properties or Git files like git.properties or through any environment property under the key info. You’ll learn how to tweak the output of this endpoint in the next section.

By default, only the health and info endpoints are exposed over HTTP. That’s why the /actuator page lists only the health and info endpoints.
```
info.app.myproperty="Property description"
```

### How do you change logging level of a package using loggers endpoint?
To configure a given logger, POST a partial entity to the resource’s URI, as shown in the following example:
```
{
    "configuredLevel": "DEBUG"
}
```

### How do you access an endpoint using a tag?
You can also add any number of tag=KEY:VALUE query parameters to the end of the URL to dimensionally drill down on a meter, e.g. /actuator/metrics/jvm.memory.max?tag=area:nonheap

### What is metrics for?
Spring Boot Actuator provides dependency management and auto-configuration for Micrometer, an application metrics facade that supports numerous monitoring systems, like AppOptics, Atlas, Datadog, Dynatrace, Elastic, Ganglia

Spring Boot auto-configures a composite MeterRegistry and adds a registry to the composite for each of the supported implementations that it finds on the classpath. Having a dependency on micrometer-registry-{system} in your runtime classpath is enough for Spring Boot to configure the registry.

### How do you create a custom metric with or without tags?
To register custom metrics, inject MeterRegistry into your component, as shown in the following example:
```
class Dictionary {
    private final List<String> words = new CopyOnWriteArrayList<>();
    Dictionary(MeterRegistry registry) {
        registry.gaugeCollectionSize("dictionary.size", Tags.empty(), this.words);
    }
    // …
}
```

### What is Health Indicator? 
A HealthIndicator provides actual health information, including a Status

### What are the Health Indicators that are provided out of the box?
```
CassandraHealthIndicator
DiskSpaceHealthIndicator
DataSourceHealthIndicator
ElasticsearchHealthIndicator
JmsHealthIndicator
MailHealthIndicator
MongoHealthIndicator
RabbitHealthIndicator
RedisHealthIndicator
SolrHealthIndicator
```

### What are the Health Indicator statuses that are provided out of the box 
DOWN SERVICE_UNAVAILABLE (503)
OUT_OF_SERVICE SERVICE_UNAVAILABLE (503)
UP No mapping by default, so http status is 200
UNKNOWN No mapping by default, so http status is 200

### How do you change the Health Indicator status severity order?
management.health.status.order=FATAL, DOWN, OUT_OF_SERVICE, UNKNOWN, UP

## Spring Boot Testing
### Why do you want to leverage 3rd-party external monitoring system? Spring Boot Testing 
### When do you want to use @SpringBootTest annotation?
Integration testing. By default, @SpringBootTest  does not start a server. Spring Boot provides a @SpringBootTest annotation which can be used as an alternative to the standard spring-test @ContextConfiguration annotation when you need Spring Boot features. The annotation works by creating the ApplicationContext used in your tests via SpringApplication.

You can use the webEnvironment attribute of @SpringBootTest to further refine how your tests will run:
```
MOCK — Loads a WebApplicationContext and provides a mock servlet environment. Embedded servlet containers are not started when using this annotation. If servlet APIs are not on your classpath this mode will transparently fallback to creating a regular non-web ApplicationContext. Can be used in conjunction with @AutoConfigureMockMvc for MockMvc-based testing of your application.
RANDOM_PORT — Loads an EmbeddedWebApplicationContext and provides a real servlet environment. Embedded servlet containers are started and listening on a random port.
DEFINED_PORT — Loads an EmbeddedWebApplicationContext and provides a real servlet environment. Embedded servlet containers are started and listening on a defined port (i.e from your application.properties or on the default port 8080).
NONE — Loads an ApplicationContext using SpringApplication but does not provide any servlet environment (mock or otherwise).
```

### What does @SpringBootTest auto-configure?
```
Uses SpringBootContextLoader as the default ContextLoader when no specific @ContextConfiguration(loader=...) is defined.
Automatically searches for a @SpringBootConfiguration when nested @Configuration is not used, and no explicit classes are specified.
Allows custom Environment properties to be defined using the properties attribute.
Allows application arguments to be defined using the args attribute.
Provides support for different webEnvironment modes, including the ability to start a fully running web server listening on a defined or random port.
Registers a TestRestTemplate and/or WebTestClient bean for use in web tests that are using a fully running web server.
```

### What dependencies does spring-boot-starter-test brings to the classpath? 
JUnit 5 (including the vintage engine for backward compatibility with JUnit 4: The de-facto standard for unit testing Java applications.

Spring Test & Spring Boot Test: Utilities and integration test support for Spring Boot applications.

AssertJ: A fluent assertion library.

Hamcrest: A library of matcher objects (also known as constraints or predicates).

Mockito: A Java mocking framework.

JSONassert: An assertion library for JSON.

JsonPath: XPath for JSON.

### How do you perform integration testing with @SpringBootTest for a web application? 
Spring-Boot provides an @SpringBootTest annotation which provides spring-boot features over and above of the spring-test module. This annotation works by creating the ApplicationContext used in our tests through SpringApplication. It starts the embedded server, creates a web environment and then enables @Test methods to do integration testing.

By default, @SpringBootTest  does not start a server. We need to add attribute webEnvironment to further refine how your tests run. It has several options:
```
MOCK(Default): Loads a web ApplicationContext and provides a mock web environment

RANDOM_PORT: Loads a WebServerApplicationContext and provides a real web environment. The embedded server is started and listen on a random port. This is the one should be used for the integration test

DEFINED_PORT: Loads a WebServerApplicationContext and provides a real web environment.
```

NONE: Loads an ApplicationContext by using SpringApplication but does not provide any web environment
### When do you want to use @WebMvcTest? What does it auto-configure?
Annotation that can be used for a Spring MVC test that focuses only on Spring MVC components.
Using this annotation will disable full auto-configuration and instead apply only configuration relevant to MVC tests (i.e. @Controller, @ControllerAdvice, @JsonComponent, Converter/GenericConverter, Filter, WebMvcConfigurer and HandlerMethodArgumentResolver beans but not @Component, @Service or @Repository beans).

By default, tests annotated with @WebMvcTest will also auto-configure Spring Security and MockMvc (include support for HtmlUnit WebClient and Selenium WebDriver). For more fine-grained control of MockMVC the @AutoConfigureMockMvc annotation can be used.

Typically @WebMvcTest is used in combination with @MockBean or @Import to create any collaborators required by your @Controller beans.

### What are the differences between @MockBean and @Mock?
@MockBean will mock and replace any existing bean of the same type in the application context. If no bean of the same type is defined, a new one will be added.

@Mock is a shorthand for the Mockito.mock() method. As well, we should only use it in a test class. Unlike the mock() method, we need to enable Mockito annotations to use this annotation.

We can do this either by using the MockitoJUnitRunner to run the test or calling the MockitoAnnotations.initMocks() method explicitly.
```
@RunWith(MockitoJUnitRunner.class)
public class ServiceTest {
     
    @Mock
    Service service;
```

### When do you want @DataJpaTest for? What does it auto-configure?
With the @DataJpaTest annotation, Spring Boot provides a convenient way to set up an environment with an embedded database to test our database queries against.To test Spring Data JPA repositories, or any other JPA-related components for that matter, Spring Boot provides the @DataJpaTest annotation. 
@DataJpaTest is used to test JPA repositories. It is used in combination with @RunWith(SpringRunner.class). The annotation disables full auto-configuration and applies only configuration relevant to JPA tests. By default, tests annotated with @DataJpaTest use an embedded in-memory database.

# Resources for the above content
### http://springcertified.com/2018/12/01/how-are-you-going-to-create-an-applicationcontext-in-an-integration-test-test/
### https://spring.io/docs/reference
### https://www.ivankrizsan.se/2018/06/19/new-book-core-spring-5-certification-in-detail/
