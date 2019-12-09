# Spring Aop 面向切面编程

切面能帮助我们模块化**横切**关注点。我们在一个地方定义通用的功能，通过声明的方法定义功能以何种方式在何处应用，而无需修改受影响的的类。横切关注点可以被模块化为特殊的类，这些类被称为切面（aspect）。

## 1. 简单的AOP例子

1.1 定义了一个简单接口

```java
package concert;
public interface Performance(){
    public void perform();
}
```

1.2 定义一个实现类

```java
@Component
public class MyPerformance implements Performance {
    @Override
    public void perform(){
        System.out.println("invoke perform()");
    }
}
```

1.3 定义一切面对上面这个接口实现bean(MyPerformance)进行增强(拦截)

```java
package aop;

@Aspect // 定义切面
public class Audience {
    @PointCut("execution(** concert.Performance.perform(..))") // 定义切点
    public void pointCut(){}
    
    // 调用前拦截
    @Before("pointCut()") 
    public void beforeHandle(){ // 定义通知
        System.out.println("before handle.");
    }
    // 调用后拦截
    @After("pointCut()")
    public void afterHandle(){ // 定义通知
        System.out.println("after handle.");
    }
    // 调用后正确返回拦截
    @AfterReturning("pointCut()")
    public void afterReturningHandle(){ // 定义通知
        System.out.println("after returning handle.");
    }
    // 调用后抛出异常拦截
    @AfterThrowing("pointCut()")
    public void afterThrowingingHandle(){ // 定义通知
        System.out.println("after throwing handle.");
    }
}

```

1.4 开启AspectJ自动代理

```java
@Configuration
@EnableAspectJAutoProxy // 在一个类上声明就可以，一般在主配置类上声明
@ComponentScan
public void ConcertConfig {
    @Bean
    public Audience audience(){ // 创建这个AOP切面bean
        return new Audience();
    }
}
```

1.5 测试验证

```java
@RunWith(SpringJunit4ClassRunner.class)
@ContextConfiguration(class=ConcertConfig.class)
public class AopTest {
    @Autowired
    private Performance performance;
    
    @Test
    public void test(){
        this.performance.perform();
    }
}
```

通过上面的例子，Audience SpringBean会对MyPerformance SpringBean增强(拦截)，当你调用通过ApplicationContext获取的Performance bean，并调用performance.perform()，其会输出如下：

before handle.

invoke perform()

after returning handle.

上面的输出，说明增强正确执行了。

