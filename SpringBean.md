Spring Bean   
==========  

Bean的生命周期  
----------  

https://github.com/zhangdberic/spring/blob/master/doc/1528637-20181105211348802-396856434.png

正如你所见，在Bean 准备就绪之前，Bean 工厂执行了若干启动步骤。我们对图1.5进行详细描述。

1　Spring 对Bean 进行实例化。

2　Spring 将值和Bean 的引用注入进Bean 对应的属性中。

3　 如果Bean 实现了BeanNameAware 接口，Spring 将Bean 的ID 传递给set-BeanName() 接口方法。

4　 如果Bean 实现了BeanFactoryAware 接口，Spring 将调用setBeanFactory()接口方法，将BeanFactory 容器实例传入。

5　 如果Bean 实现了ApplicationContextAware 接口，Spring 将调用setApplicationContext()接口方法，将应用上下文的引用传入。

6　 如果Bean 实现了BeanPostProcessor 接口，Spring 将调用它们的post-ProcessBeforeInitialization() 接口方法。

7　 如果Bean 实现了InitializingBean 接口，Spring 将调用它们的after-PropertiesSet() 接口方法。类似地，如果Bean 使用init-method 声明了初始化方法，该方法也会被调用。

8　 如果Bean 实现了BeanPostProcessor 接口，Spring 将调用它们的post-PoressAfterInitialization() 方法。

9　 此时此刻，Bean 已经准备就绪，可以被应用程序使用了，它们将一直驻留在应用上下文中，直到该应用上下文被销毁。

10　 如果Bean 实现了DisposableBean 接口，Spring 将调用它的destroy()接口方法。同样，如果Bean 使用destroy-method 声明了销毁方法，该方法也会被调用。



