# 基于JavaConfig创建spring应用上下文

Spring从两个角度来实现自动化装配：

1. 组件扫描（Component scanning）：Spring会自动发现应用上下文中所创建的bean。

2. 自动装配（autowiring）：Spring自动满足bean之间的依赖关系。

组件扫描和自动装配组合在一起就能发挥出强大的威力，它们能够将你的显示配置降低到最少。

## 1. JavaConfig方式创建Spring Bean

基于JavaConfig方式创建Spring bean有两种方式：

1. 通过@Component声明类，并使用@ComponentScan能扫描到这个类，这个类就是一个Spring Bean。

2. 通过@Bean声明的方法，其返回值对象就是一个Spring Bean。

当然也可以混合以上两种方式来创建SpringBean，甚至再混合XML配置来共同创建SpringBean。

### 1.1 基于@Component+@ComponentScan组合来创建SpringBean

#### 1.1.1 @Component

基于@Component源注释声明的类

```java
package com.spring.javaconfig.springbean;
@Component
public class UserLogic {
    public User addUser(User user){
        return new User(1,user);
    }
}
```

注意：@Component声明的类只有被@ComponentScan扫描到才能称为Spring bean，才能装配到Spring上下文中。

在Spring中为了更好理解类的作用，在@Component的基础上有声明了@Service、@Controller、@Respository等源注释。

#### 1.1.2 @ComponentScan

声明为@ComponentScan的配置类，Spring应用上下文会扫描指定包(package)下声明为@Component的类创建Spring Bean。

```java
package com.spring.javaconfig;

@Configuration
@ComponentScan
public class BeanConfig {
}

```

如果没有显示的配置@ComponentScan，@ComponentScan默认会扫描与配置类相同的包和其下的所有子包，查找带有@Component注释的类。如上的例子：会扫描BeanConfig类所在包(com.spring.javaconfig)和其下子包(com.spring.javaconfig.springbean)，会扫描到UserLogic类并创建为SpringBean。

也可以配置@ComponentScan源注释，扫描指定包，例如：

@ComponentScan(backPackage={"com.sc.logic","com.sc.controller"}) #  扫描指定包名下的@Component类

@ComponentScan(backPackageClasses={Logic.class,Controller.class}) # 扫描类所在包下的@Component类，例如：会扫描Logic.class所在的包和Controller.class所在包，在实际开发中，可以创建类空标记接口类，专门用于bean扫描。

#### 1.1.3 AnnotationConfigApplicationContext

基于AnnotationConfigApplicationContext来创建Spring应用上下文，例如：

```java
package com.spring;
import com.spring.javaconfig;
import com.spring.javaconfig.springbean;

public class Application {
    public static void main(String[] args){
        ApplicationContext context = new AnnotationConfigApplicationContext(com.spring.javaconfig.BeanConfig.class);
        BeanConfig beanConfig = context.getBean(BeanConfig);        
    }
}

```

AnnotationConfigApplicationContext构造方法参数是使用@Configuration声明的配置类，可以是多个(数组)，其会加载这个配置类，根据配置类上声明的@ComponentScan来扫描指定包下的@Component类，创建springBean。

#### 1.1.4 测试用例

```java
package com.spring.javaconfig.test;
import com.spring.javaconfig;
import com.spring.javaconfig.springbean;

@RunWith(SpringJunit4ClassRunner.class)
@ContextConfiguration(class=BeanConfig.class)
public class UserLogic {
    @Autowired
    private UserLogic userLogic;
    
    @Test
    public void addUser(){
        this.userLogic.addUser(new User(1));
    }
}
```

@ContextConfiguration(class=BeanConfig.class)，配置了使用的配置类。



### 1.2 @Bean 创建SpringBean

#### 1.2.1 @Bean

基于@Bean源注释声明的方法，创建SpringBean

```java
package com.spring.javaconfig;

@Configuration
public class BeanConfig {
    @Bean
    public UserLogic userLogic(){
        return new UserLogic();
    }
}

```

注意：@Bean声明的方法所在的类，应该使用@Configuration源注释声明为配置类。

**@Bean定义配置清除方法**

@Bean(destroyMethd="close")



#### 1.2.2 AnnotationConfigApplicationContext

基于AnnotationConfigApplicationContext来创建Spring应用上下文，例如：

```java
package com.spring;
import com.spring.javaconfig;
import com.spring.javaconfig.springbean;

public class Application {
    public static void main(String[] args){
        ApplicationContext context = new AnnotationConfigApplicationContext(com.spring.javaconfig.BeanConfig.class);
        BeanConfig beanConfig = context.getBean(BeanConfig);        
    }
}

```

AnnotationConfigApplicationContext构造方法参数是使用@Configuration声明的配置类，可以是多个(数组)，其会加载这个配置类，根据配置类上声明的@Bean来创建springBean。

#### 1.2.3 测试用例

```java
package com.spring.javaconfig.test;
import com.spring.javaconfig;
import com.spring.javaconfig.springbean;

@RunWith(SpringJunit4ClassRunner.class)
@ContextConfiguration(class=BeanConfig.class)
public class TestUserLogic {
    @Autowired
    private UserLogic userLogic;
    
    @Test
    public void addUser(){
        this.userLogic.addUser(new User(1));
    }
}
```

@ContextConfiguration(class=BeanConfig.class)，配置了使用的配置类。

### 1.3 Configuration配置类

基于@Configuration源注释声明配置类

```java
@Configuration
@Import(LogicConfig.class)
@ImportResource("classpath:Controller-Config.xml")
public class AppConfig{

}
```

@Import可以引用其他的@Configuration声明的配置类，@ImportResource可以引用xml方式的bean配置。



## 2. 自动装配(注入)

基于@Component方式，可以使用@Autowired注入。基于@Bean方式，可以使用JavaConfig本地方法和参数引用注入。当然可以混合使用两种方式，来注入，例如：使用@Bean创建的SpringBean，使用@Autowired来注入、使用@Component声明的SpringBean，使用JavaConfig的本地方法和参数引用来注入。

### 2.1 @Autowired

#### 2.1.1 @Autowired

```java
@Component
public class UserService {
    // 基于属性方式注射Bean
    @Autowired 
    private UserLogic userLogic;
    // 基于属性方式注射Bean，但允许为null
    @Autowired(required=false)
    private RoleLogic roleLogic;
    
    private GroupLogic groupLogic;
    
    // 基于构造方法注射Bean
    @Autowired
    public UserService(LoggerLogic loggerLogic) {
        
    }
    // 基于方法参数来注射Bean
    @Autowired
    private void setGroupLogic(GroupLogic groupLogic){
        this.groupLogic = groupLogic;
    }
}
```

#### 2.1.2 解决歧义性

如果有多bean能够匹配，这种歧义性会阻碍Spring自动装配性。

例如：一个接口多个bean实现

```java
@Autowired
private void setDessert(Dessert dessert){
    this.dessert = dessert;
}

@Component
public class Cake implements Dessert {...}
@Component
public class Cookie implements Dessert {...}
@Component
public class IceCream implements Dessert {...}

```

因为三个Bean都实现了Dessert，当Spring试图装配setDessert()中的Dessert参数时，不是唯一，有歧义可选性，Spring会抛出NoUnqiueBeanDefinitionException异常。

#### 2.1.2.1 @Qualifier

如果有两个相同类(相同的Class)，创建了两个SpringBean，而@Autowired要依赖于其中的一个，那么就要显示的使用@Qualifier("xxx")来声明依赖其中一个。

@Qualifier("bean名称")

```java
    @Autowired
    @Qualifier("cookie")
    private void setDessert(Dessert dessert){
        this.dessert = dessert;
    }
```

例如：@Qualifier("cookie")，这里显示的制定了使用cookie的springBean。

#### 2.1.2.2 @Primary

@Autowired+@Qualifier配合指定装配的SpringBean名称，@Component+@Primary配合指定了在出现多种可装配的SpringBean，也就是发现歧义性的时候，默认使用哪个SpringBean。

```java
@Component
@Primary
public class Cookie implements Dessert {...}
```



#### 2.1.2 JavaConfig @Bean(注入)

```java
@Configuration
public class UserConfig {
    
    @Bean
    public UserLogic userLogic() {
        return new UserLogic();
    }
    
    @Bean
    public UserService userService(){
        // 同一个配置类对象内部，通过本地方法实现bean注入
        return new UserService(userLogic());
    }
}

    
```

注意：userLogic()方法始终返回同一个UserLogic对象，userLogic()方法内代码只能被执行一次，保证UserLogic的单实例。原理：基于CGLIB代理，其会判断是否已经创建了UserLogic。

```java
@Configuration
public class GroupConfig {
	// 不同配置类，通过参数引用实现bean注入
    @Bean
    public GroupService groupService(UserLogic userLogic){
        return new GroupService(userLogic);
    }
}
```









