# Spring面试题

## 什么是三级缓存

1. 第一级缓存：单例缓存池singletonObjects。
2. 第二级缓存：早期提前暴露的对象缓存earlySingletonObjects。（属性还没有值对象也没有被初始化）
3. 第三级缓存：singletonFactories单例对象工厂缓存。

## Spring如何解决循环依赖问题

- Spring使用了三级缓存解决了循环依赖的问题。
- 在populateBean()给属性赋值阶段里面Spring会解析你的属性，并且赋值，当发现，A对象里面依赖了B，此时又会走getBean方法，但这个时候，你去缓存中是可以拿的到的。
- 因为我们在对createBeanInstance对象创建完成以后已经放入了缓存当中，所以创建B的时候发现依赖A，直接就从缓存中去拿，此时B创建完，A也创建完，一共执行了4次。至此Bean的创建完成，最后将创建好的Bean放入单例缓存池中。