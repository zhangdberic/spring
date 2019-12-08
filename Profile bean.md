# 配置Profile Bean

Spring根据指定的环境来创建@Bean，例如：根据dev环境来创建H2数据库Datasource，根据prod来创建oracle数据库Datasource，然后在开发环境指定dev，运行环境指定prod环境。



## 1. @Profile源注释

@Profile源注释可以应用在类(class)级别和方法(method)级别上，可以根据项目情况来使用，例如：如果应用在类级别上，则整个类为Profile类，你可以定义了DevelopmentProfileConfig、ProdProfileConfig等，把环境差异的Bean使用不同的配置类区分开。

### 1.1 @Profile注释应用类(class)级别

@Profile("xxx)注释应用在类级别上，则这个类内所有@Bean方法，都只在xxx环境下才有效。

```java
@Configuration
@Profile("dev")
public class DevelopmentProfileConfig {
    @Bean
    public DataSource dataSource(){
        return new H2DataSource();
    }
    @Bean
    public Cache cache(){
        return new MapCache();
    }
}
```

例如：上面例子，只有在dev环境下启动，spring上下文件才会创建H2Datasource和MapCache两个Spring Bean。

### 1.2 @Profile注释应用方法(method)级别

@Profile("xxx")注释应用在方法级别上，则这个@Bean方法，只有在xxx环境下才有效。

```java
@Configuration
public class DevelopmentProfileConfig {

    @Bean
    @Profile("dev")
    public DataSource dataSource(){
        return new H2DataSource();
    }
    
    @Bean
    @Profile("prod")
    public DataSource dataSource(){
        return new Dbcp2DataSource();
    }
}  
   
```

例如：上面例子，如果dev环境启动，则H2DataSource Bean有效。如果prod环境启动，则Dbcp2DataSource Bean有效。

## 2. 激活Profile

上面的@Profile源注释，定义了那些环境(@Profile("xxx"))创建那些Bean。因为需要在系统启动的时候指定当前所处的环境。

spring.profiles.active=xxx，指定了当前运行环境。如果没有指定这个属性，则使用spring.profiles.default属性值。如果上面两个属性都没有配置，则所有@Profile("xxx")注释的bean都不会被创建。

有多种方式来设置这两个值：

系统环境变量； linux 的系统环境变量

**jvm系统属性；  例如：-Dspring.profiles.active=prod -Dspring.profiles.default=dev**

集成测试@ActiveProfiles注释；

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(class={PersistenceTestConfig.class})
@ActiveProfiles("dev")
public class PersistenceTest {
    
}
```











