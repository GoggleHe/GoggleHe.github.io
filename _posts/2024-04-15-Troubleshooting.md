---
layout: post
title: 线上问题排查定位
categories: [Java]
description: 线上问题排查定位
keywords: Java, Linux
---

## 线上问题排查定位

### CPU 100%

#### 模拟代码

```java
import java.util.Random;

public class Loop {
    public static void main(String[] args) {
        cpuTest();
    }

    private static void cpuTest() {
        new Thread(() -> {
            while (true) {
                new Object();
            }
        }, "CPU-100").start();
        while (true) {
            for (int i = 0; i < 10; i++) {
                new Thread(() -> {
                    try {
                        new Object();
                        long random = new Random().nextInt(200);
                        Thread.sleep(random);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }).start();
            }
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

```

#### 排查

top命令定位进程

top -H -p 定位线程

jstack 获取线程信息日志

分析日志

### 死锁

#### 模拟代码

```java
public class DeadlockExample {
    private static final Object lockA = new Object();
    private static final Object lockB = new Object();

    public static void main(String[] args) {
        // 启动线程1，尝试首先获取lockA然后lockB
        Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (lockA) {
                    System.out.println("Thread 1: Holding lock A...");
                    try {
                        Thread.sleep(1000); // 等待一段时间以确保thread2可以执行并且获取lockB
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    synchronized (lockB) {
                        System.out.println("Thread 1: Holding lock B...");
                    }
                }
            }
        });
        thread1.start();

        // 启动线程2，尝试首先获取lockB然后lockA
        Thread thread2 = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (lockB) {
                    System.out.println("Thread 2: Holding lock B...");
                    try {
                        Thread.sleep(1000); // 等待一段时间以确保thread1可以执行并且获取lockA
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    synchronized (lockA) {
                        System.out.println("Thread 2: Holding lock A...");
                    }
                }
            }
        });
        thread2.start();
    }
}
```

#### 排查