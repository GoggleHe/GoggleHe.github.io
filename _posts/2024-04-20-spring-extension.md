---
layout: post
title: Spring扩展点汇总
categories: [Spring]
description: Spring扩展点汇总
keywords: Java, Spring
---

## Spring扩展点汇总

### 1、Aware系列

在Spring Boot中，有一些可以实现的Aware接口，用于在应用程序中获取特定的上下文或对象。这些接口允许您的组件意识到它们所在的环境，并与之进行交互。以下是在Spring Boot中常见的Aware接口：

#### 1.1 ApplicationContextAware

通过实现该接口，您的组件可以获取ApplicationContext（应用程序上下文）对象的引用，从而访问应用程序上下文中的Bean以及其他特定的Spring功能。

#### 1.2 BeanFactoryAware

实现该接口，您的组件可以获取BeanFactory（Bean工厂）对象的引用。这允许您在运行时从Bean工厂中获取其他Bean。

#### 1.3 EnvironmentAware

通过实现该接口，您的组件可以获取Environment（环境）对象的引用，从而访问应用程序的配置属性和配置文件。

#### 1.4 MessageSourceAware

实现该接口，您的组件可以获取MessageSource（消息源）对象的引用，从而访问国际化消息。

#### 1.5 ResourceLoaderAware

通过实现该接口，您的组件可以获取ResourceLoader（资源加载器）对象的引用，从而加载外部资源文件。

#### 1.6 ServletContextAware

实现该接口，您的组件可以获取ServletContext（Servlet上下文）对象的引用，从而访问与Web应用程序相关的功能。

这些接口都属于Spring框架的一部分，可以通过在您的组件类中实现相应的接口并实现相应的方法来使用它们。这样，当您的组件在Spring容器中创建时，Spring会自动将适当的上下文或对象引用注入到您的组件中，以便您可以使用它们。

### 2、Bean的生命周期扩展接口

在Spring框架中，您可以使用以下接口来扩展Bean的生命周期：

#### 2.1InitializingBean

通过实现InitializingBean接口，您的Bean可以在初始化阶段执行自定义逻辑。该接口包含一个afterPropertiesSet()方法，您可以在此方法中定义初始化逻辑。

#### 2.2 DisposableBean

通过实现DisposableBean接口，您的Bean可以在销毁阶段执行自定义逻辑。该接口包含一个destroy()方法，您可以在此方法中定义销毁逻辑。

#### 2.3 @PostConstruct

使用@PostConstruct注解可以在Bean的初始化阶段指定一个方法。该方法将在依赖注入完成后立即执行。

#### 2.4 @PreDestroy

使用@PreDestroy注解可以在Bean销毁之前指定一个方法。该方法将在Bean销毁前执行。

#### 2.5 BeanPostProcessor

BeanPostProcessor接口定义了在容器实例化Bean之后和初始化之前，对Bean进行自定义处理的方法。通过实现该接口，您可以插入自定义逻辑来处理Bean。

#### 2.6 BeanFactoryPostProcessor

BeanFactoryPostProcessor接口允许在所有Bean定义加载到容器之后，但在Bean实例化之前对它们进行自定义处理。通过实现该接口，您可以修改或添加新的Bean定义。

#### 2.7 BeanDefinitionRegistryPostProcessor

这是BeanFactoryPostProcessor接口的扩展，允许在Bean定义注册过程中对Bean定义进行自定义处理。它提供了对Bean定义注册表的直接访问，可以添加、删除或修改Bean定义。

这些接口提供了不同层面的扩展点，可以让您在Bean的生命周期中插入自定义逻辑。您可以根据需要选择适合您的场景的接口来实现自定义的Bean生命周期行为。

### 3、内置变量

Spring Boot有一些内置变量可供在应用程序中使用。以下是其中一些常用的内置变量：

- ${random.*}：生成随机值的变量，例如${random.int}生成一个随机整数。
- ${server.*}：获取与服务器相关的属性，如${server.port}获取应用程序正在运行的端口。
- ${spring.*}：获取Spring Boot配置的属性，如${spring.application.name}获取应用程序的名称。
- ${local.*}：获取本地机器的相关属性，如${local.ip-address}获取本地IP地址。
- ${application.*}：获取应用程序特定的属性，需要在配置文件中定义，如${application.custom-property}获取自定义属性。

除了以上内置变量，您还可以在应用程序中使用自定义的环境变量或在配置文件中定义的属性。


### 4、web扩展点

#### 4.1 HandlerMethodArgumentResolver

实现HandlerMethodArgumentResolver接口可以创建自定义的方法参数解析器。方法参数解析器用于将请求参数映射到控制器方法的参数上。

#### 4.2 HandlerMethodReturnValueHandler

实现HandlerMethodReturnValueHandler接口可以创建自定义的方法返回值处理器。方法返回值处理器用于将控制器方法的返回值转换为响应的内容。

#### 4.3 CorsConfigurationSource

通过实现CorsConfigurationSource接口，您可以自定义跨域资源共享（CORS）的配置。可以在该接口中设置允许的来源、方法和头部等。

#### 4.4EmbeddedValueResolverAware

通过实现EmbeddedValueResolverAware接口，您的组件可以获取EmbeddedValueResolver对象的引用，用于解析字符串中的占位符和表达式。

#### 4.5 ViewResolver

通过实现ViewResolver接口，您可以创建自定义的视图解析器。视图解析器用于将逻辑视图名称解析为实际的视图实现，例如JSP、Thymeleaf模板等。

### 5、应用生命周期扩展点

使用以下扩展点来扩展应用程序的生命周期：

#### 5.1 SpringApplicationRunListener

通过实现SpringApplicationRunListener接口，您可以在应用程序启动的不同阶段插入自定义逻辑。该接口定义了多个方法，例如在应用程序启动前、启动成功后、出现异常时等情况下执行的回调方法。

#### 5.2 ApplicationRunner

和CommandLineRunner：这两个接口在前面已经提到过，它们允许您在应用程序启动后执行特定的逻辑。您可以实现其中一个或两者来定义需要在应用程序启动后立即执行的操作。

#### 5.3 ContextRefreshedEvent

和ContextClosedEvent：这些是Spring框架中的事件类，您可以监听并在应用程序上下文刷新或关闭时执行相应的操作。通过实现ApplicationListener或ApplicationListener接口，并处理对应事件的回调方法，可以在应用程序生命周期的特定点添加自定义逻辑。

#### 5.4 SmartLifecycle

实现SmartLifecycle接口可以创建一个具有更精细控制的组件，它可以在应用程序启动时自动启动，并在关闭时自动停止。该接口定义了多个方法，例如控制启动顺序、判断是否要自动启动和停止等。

#### 5.5 ShutdownHook

Spring Boot应用程序在关闭时会注册一个JVM关闭钩子。您可以使用SpringApplication.addShutdownHook()方法注册自定义的关闭钩子，以执行一些清理或释放资源的操作。

通过使用这些扩展点，可以在应用程序的不同生命周期阶段插入自定义逻辑，例如应用程序启动前的准备、应用程序启动后的初始化、应用程序关闭时的清理等。这些扩展点提供了更大的灵活性，使您能够定制和控制应用程序的整个生命周期。
