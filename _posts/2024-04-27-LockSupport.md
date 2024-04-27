---
layout: post
title: LockSupport
categories: [多线程]
description: LockSupport
keywords: LockSupport, Java
---

## LockSupport

- LockSupport内部维持一个permit，取值0/1,初始为0，表示“许可”。
- park()等待许可，unpark()提供许可。park()消费permit为0，unpark()提供permit为1。
- unpark()可以在park()之前
- Thread.interrupt()设置中断标志为true，并调用unpark()，可以将park()唤醒，但不抛出异常。
- 若中断标志为true，后续执行的park()都不会阻塞，也不会消费permit。
- 等遇到Thread.sleep()时，异常才会抛出或被捕获。
- 一次interrupt可以中断一次sleep，中断标志被sleep消费，还可以中断一次park，permit被park消费。