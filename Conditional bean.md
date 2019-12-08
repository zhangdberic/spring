# @Conditional 条件化Bean

Spring4引入了一个新的@Conditional注释，它可以用到带有@Bean注释的方法上。如果给定的条件计算结果为true，就会创建这个bean，否则的话，这个bean会被忽略。这个特性非常重要，其是Spring Boot基石。



## @Conditional

**@Conditional(条件类)**，条件类返回true，则创建这个Bean。

这里的条件类，是一个接口(Condition)实现类，如果这个接口的匹配方法(matches)返回true，则创建这个Bean，接口代码如下。

```java
public interface Condition {
    boolean matches(ConditionContext ctxt,AnnotatedTypeMetadata metadata);
}
```

例如：

```java
@Bean
@Conditional(MagicExistsCondition.class)
public MagicBean magicBean(){
    return new MagicBean();
}
```

```java
public class MagicExistsCondition implements Condition {
    public boolean matches(ConditionContext ctxt,AnnotatedTypeMetadata metadata) {
        // 根据环境变量中是否有"magic"属性来创建bean
        Environment env = context.getEnvironment();
        return env.containsProperty("magic");
    }
}
```



## ConditionalContext参数

ConditionalContext是一个接口，通过ConditionalContext，我们可以得到如下：

```java
public interface ConditionContext {
    BeanDefinitionRegistry getRegistry(); // 用于检查Bean的定义
    ConfigurableListableBeanFactory getBeanFactory(); // 用于检查Bean是否存在,获取bean属性
    Environment getEnvironment(); // 用于检查环境变量
    ResourceLoader getResourceLoader(); // 获取资源加载器，判断资源是否存在，根据资源内容来判断
    ClassLoader getClassLoader(); // 返回classLoader，用于检测类是否存在
}
```



## AnnotatedTypeMetadata参数

AnnotatedTypeMetadata也是一个接口，通过AnnotatedTypeMetadata，我们可以得到如下：

```java
public interface AnnotatedTypeMetadata {

    boolean isAnnotated(String annotationType); // 这个@Bean方法上是否还有其它源注释
    Map<String, Object> getAnnotationAttributes(String annotationType); // 获取这个@Bean方法上某个源注释的属性值
    Map<String, Object> getAnnotationAttributes(String annotationType, boolean classValuesAsString);

    MultiValueMap<String, Object> getAllAnnotationAttributes(String annotationType);
    MultiValueMap<String, Object> getAllAnnotationAttributes(String annotationType, boolean classValuesAsString);

}
```

例如：

```java
class ProfileCondition implements Condition {

    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        if (context.getEnvironment() != null) {
            MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName()); // 获取@Bean方法上的@Profile注释的属性值
            if (attrs != null) {
                for (Object value : attrs.get("value")) {
                    if (context.getEnvironment().acceptsProfiles(((String[]) value))) {
                        return true;
                    }
                }
                return false;
            }
        }
        return true;
    }

}
```

