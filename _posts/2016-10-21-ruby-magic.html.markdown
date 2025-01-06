---
title: Ruby
date: 2016-10-21
tags: Ruby
---

### 排列组合
```
arr = [1, 2, 3, 4, 5]
arr.combination(3).to_a
ary.each_cons(3).to_a
```
### ruby GIL
```
1: Ruby MRI为了线程安全，用了GIL只支持单核。任何时刻只有一个线程在运行：当一个Thread阻塞时（比如IO, sleep等），就会切换到另个一线程。 所以就算用了celluloid, puma自动创建多线程的，MRI下每个进程还是单核。
而Rubinius，JRuby则没有GIL，支持多核（每个线程分配一个cpu）。
2: 如果MRI下要用多核，就要用sidekiq之类的跑多个worker进程。
3: 单核不一定比多核慢，很多时候瓶颈不在CPU，而MRI多线程可以保证IO并发，所以性能可能是一样的。
4: 多进程消耗内存，而且进程间难以通信(只能通过IO之类的)。多线程，直接共享变量（内存）。
5: 在 IO 等待比较多的场景中，应用多线程与多进程都能不同程度的提高效率
6: 在 CPU 计算比较多的场景中，多线程对程序的效率提升随着计算量的提升，几乎可以忽略不计
```
### ruby多线程-多进程
1. EventMachine
2. MultiThread
3. Celluloid

### wicked_pdf中文乱码解决
```
apt-get install ttf-wqy-zenhei
```
