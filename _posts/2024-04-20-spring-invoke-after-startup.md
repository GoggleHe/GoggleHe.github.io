---
layout: post
title: Spring容器启动后调用
categories: [Spring]
description: Spring容器启动后调用
keywords: Java, Spring
---

## Spring容器启动后调用

### CommandLineRunner

```java
import org.springframework.boot.CommandLineRunner;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

@Component
@Order(2)
public class MyCommandLineRunner implements CommandLineRunner {

    @Override
    public void run(String... args) throws Exception {
        System.out.println("this is MyCommandLineRunner executed.");
    }
}

```

### ApplicationRunner

```java
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

 
@Component
@Order(1)
public class MyApplicationRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {

        System.out.println("this is MyApplicationRunner executed.");
    }
}
```

### ApplicationListener

```java
/**
* Spring事件监听：使用ApplicationListener监听ContextRefreshedEvent事件
*/
@Component
public class MyApplicationListener implements ApplicationListener<ContextRefreshedEvent> {
    @Autowired
    private MyService myService;

    @Override
    public void onApplicationEvent(ContextRefreshedEvent contextRefreshedEvent) {
        System.out.println(">>>>>>>>>>>>>> ApplicationListener:" + myService.sayHello());
    }
}

```

### SmartInitializingSingleton 

```java
/**
 * 
 * 
 */
@Component
public class MySmartInitializingSingleton implements SmartInitializingSingleton {
    @Autowired
    private MyService myService;

    @Override
    public void afterSingletonsInstantiated() {
        System.out.println(">>>>>>>>>>>>>> SmartInitializingSingleton:" + myService.sayHello());
    }
}
```

### SmartLifecycle

```java
/**
 * 
 */
@Component
public class MySmartLifecycle implements SmartLifecycle {

    @Autowired
    private MyService myService;

    @Override
    public void start() {
        System.out.println(">>>>>>>>>>>>>> SmartLifecycle:" + myService.sayHello());
    }

    @Override
    public boolean isAutoStartup() {
        return true;
    }

    @Override
    public void stop(Runnable callback) {

    }

    @Override
    public void stop() {

    }

    @Override
    public boolean isRunning() {
        return false;
    }

    @Override
    public int getPhase() {
        return 0;
    }
}
```

### LifeCycle

```java
@Component
public class MyLifeCycle implements Lifecycle {
    private volatile boolean running;
    @Override
    public void start() {
        running = true;
        System.out.println("start");
    }

    @Override
    public void stop() {
        running = false;
        System.out.println("stop");
    }

    @Override
    public boolean isRunning() {
        return running;
    }
}
```

### SmartLifeCycle

```java
@Component
public class MyLifeCycle implements SmartLifecycle {
    private volatile boolean running;
    @Override
    public void start() {
        running = true;
        System.out.println("start");
    }

    @Override
    public void stop() {
        running = false;
        System.out.println("stop");
    }

    @Override
    public boolean isRunning() {
        return running;
    }

    @Override
    public boolean isAutoStartup() {
        return SmartLifecycle.super.isAutoStartup();
    }

    @Override
    public void stop(Runnable callback) {
        stop();
        callback.run();
    }

    @Override
    public int getPhase() {
        return 0;
    }
}
```

