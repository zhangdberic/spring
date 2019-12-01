spring应用上下文ApplicationContext  
==========  
Spring自带了多种类型的应用上下文：  

AnnotationConfigApplicationContext：从一个或多个Java的配置类中加载Spring应用上下文。  

AnnotationConfigWebApplicationContext：从一个或多个Java的配置类中加载Spring Web应用上下文。  

ClasspathXmlApplicationContext：从类路径下一个或多个XML配置文件中加载上下配置文件。  

FileSystemApplicationContext：从文件系统下的一个或多个XML配置文件中加载上下配置文件。  

XmlWebApplicationContext：从Web应用下的一个或多个XML配置文件加载上下配置文件。  

例如：使用JavaConfig方式加载Spring应用上下文，
```java  
ApplicationContext context = new AnnotationConfigApplicationContext(com.sc.config.ApplicationConfig.class);
```  
这个com.sc.config.ApplicationConfig类声明为@Configuration源注释（配置类）








