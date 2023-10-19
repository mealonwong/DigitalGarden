---
{"dg-publish":true,"permalink":"/spring/ioc/"}
---


IOC，控制反转，是Spring的核心组件，用以保存管理Spring的所有bean对象，总而言之就是一个容器。
它负责创建、配置和管理bean，控制bean的依赖注入。通俗点讲就是我们不需要管对象的创建以及对象之间依赖注入这些，只需要直接使用就可以了。
ApplicationContext继承BeanFactory（getBean()核心方法），BeanFactory可以简单粗暴的理解为一个Map，不过是CHM。
什么是依赖注入？就是把程序运行期间所依赖的资源从外部注入到内部，维护程序内外对象的关系。

如何简单实现一个IOC？
首先需要一个容器，这里我们可以选择ConcurrentHashMap作为容器维护beanName和Bean对象之间的关系

另外需要提供一个入口，其实根据不同的加载方式可以提供多个入口，例如使用注解管理Bean对象或者使用配置文件管理Bean对象，从而提供不同的具体实现。

其次需要生成Bean对象，那么此时需要一个BeanFactory进行。

然后我们需要一个扫描器或者叫做读取器，可以对指定路径下的配置文件进行扫描或者对使用了某些注解的类进行解析，然后将生成的Bean对象注册进容器中。