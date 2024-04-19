---
layout: post
title: 多线程获取返回值的方式
categories: [多线程]
description:多线程获取返回值的方式
keywords: Java, Linux
---

## 多线程获取返回值的方式

### 无误返回值

```java
package org.example.concurrent.executor;

import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {
        ThreadPoolExecutor executor = new ThreadPoolExecutor(10, 20, 10, TimeUnit.SECONDS, new ArrayBlockingQueue<>(50));
        executor.submit(() -> {
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
    }
}

```

### 有返回值

#### Thread+Callable+FutureTask

```java
package org.example.concurrent.completefuture;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;
import java.util.concurrent.TimeUnit;

public class Main {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        Callable<String> callable = () -> {
            TimeUnit.SECONDS.sleep(3);
            return "hello world";
        };

        FutureTask<String> futureTask = new FutureTask<>(callable);
        Thread thread = new Thread(futureTask);
        thread.start();
        String s = futureTask.get();
        System.out.println(s);
    }
}

```

#### 线程池+Future+Callable

```java
package org.example.concurrent.executor;

import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {
        ThreadPoolExecutor executor = new ThreadPoolExecutor(10, 20, 10, TimeUnit.SECONDS, new ArrayBlockingQueue<>(50));
        Future<Integer> future = executor.submit(() -> {
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return 10;
        });
        try {
            Integer integer = future.get();
            System.out.println(integer);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}

```

#### 线程池+Runnable+Result+Future

```java
package org.example.concurrent.executor;

import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {
        ThreadPoolExecutor executor = new ThreadPoolExecutor(10, 20, 10, TimeUnit.SECONDS, new ArrayBlockingQueue<>(50));
        Data data = new Data();
        Future<Data> integerFuture = executor.submit(new MyTask(data), data);
        try {
            System.out.println("data=" + data);//null
            Data get = integerFuture.get();
            System.out.println("get=" + get);//有值
            System.out.println("data=" + data);//有值
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }


    }

    static class Data {
        private Integer age;
        private String name;

        public Integer getAge() {
            return age;
        }

        public void setAge(Integer age) {
            this.age = age;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        @Override
        public String toString() {
            return "Data{" +
                    "age=" + age +
                    ", name='" + name + '\'' +
                    '}';
        }
    }


    static class MyTask implements Runnable {
        private final Data data;

        public MyTask(Data data) {
            this.data = data;
        }

        @Override
        public void run() {
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            data.setAge(10000);
            data.setName("hello");
        }
    }

}

```

#### CompletionService

```java
package org.example.concurrent.completionservice;

import java.util.concurrent.*;

/**
 * CompletionService是用于管理多个future对象的辅助类
 **/
public class Main {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService fixedThreadPool = Executors.newFixedThreadPool(5);
        CompletionService<String> completionService = new ExecutorCompletionService<>(fixedThreadPool);
        for (int i = 0; i < 10; i++) {
            int finalI = i;
            Callable<String> task = () -> {
                TimeUnit.SECONDS.sleep(3);
                return "hello world! " + finalI;
            };
            completionService.submit(task);
        }
        while (true) {
            Future<String> take = completionService.take();
            String s = take.get();
            System.out.println(s);
        }
    }
}
```

#### CompletableFuture

```java
package org.example.concurrent.completefuture;

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Main {
    public static void main(String[] args) {
        ExecutorService newFixedThreadPool = Executors.newFixedThreadPool(10);
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "hello world", newFixedThreadPool);
        try {
            future.get();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```

