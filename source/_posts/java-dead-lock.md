---
title: Java死锁排查
tags:
  - JAVA
categories:
  - JAVA
  - 技术
  - 经验
toc: true
date: 2020-08-14 11:24:59
---

# 模拟死锁

```java
/**
 * 
 * Company: 锦海捷亚
 *
 * @author Sealin
 * @date 2020/8/14
 */
public class TestDeadLock {
    public static void main(String[] args) {
        Object lock1 = new Object();
        Object lock2 = new Object();
        Thread thread1 = new Thread(() -> {
            synchronized (lock1) {
                try {
                    Thread.sleep(1000);
                    synchronized (lock2) {

                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        Thread thread2 = new Thread(() -> {
            synchronized (lock2) {
                try {
                    Thread.sleep(1000);
                    synchronized (lock1) {

                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

            }
        });

        thread1.start();
        thread2.start();
    }
}

```

## 原因分析
执行以上代码, 发现程序一直在执行, 没有按预期的流程正常退出.
导致死锁的原因, 只是因为不同线程之间相互持有了所需资源, 并且都在等对方释放,自己才能继续执行后续代码. 如同两个都只拿了一根筷子的人, 都打算让对方先把手上都筷子交给自己, 自己吃完饭再把筷子交出去一样, 双方都在等待, 导致的结果就是死锁.

# 错误排查
首先找到当前应用的PID, 使用jdk提供的jps命令.
```bash
sealin@Sealin: ~ $ jps                                                                                                                                                                                
87696 TestDeadLock
87840 Jps
86931 RemoteMavenServer36
```

可以找到我们测试死锁的进程pid为: 87696
使用jstack工具查看当前进程信息
```bash
sealin@Sealin: ~ $ jstack 87696   

....省略部分内容....

===================================================
"Thread-1":
        at com.jhj.winform.server.TestDeadLock.lambda$main$1(TestDeadLock.java:33)
        - waiting to lock <0x000000071625f360> (a java.lang.Object)
        - locked <0x000000071625f370> (a java.lang.Object)
        at com.jhj.winform.server.TestDeadLock$$Lambda$2/325333723.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)
"Thread-0":
        at com.jhj.winform.server.TestDeadLock.lambda$main$0(TestDeadLock.java:20)
        - waiting to lock <0x000000071625f370> (a java.lang.Object)
        - locked <0x000000071625f360> (a java.lang.Object)
        at com.jhj.winform.server.TestDeadLock$$Lambda$1/2066940133.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.
```
![image.png](/images/2020/08/14/16ae44bd-5755-429d-86cc-524336b95248.png)
很明显的可以看到死锁发生的位置, 进入 TestDeadLock.java:33 和 TestDeadLock.java:20 查看执行的什么操作导致了死锁
![image.png](/images/2020/08/14/d0e2e766-6624-41b2-8ff4-71980db10409.png)
第33行是我们thread2对lock1进行加锁的位置
![image.png](/images/2020/08/14/59b2d73a-207c-47d9-972d-523c536ac904.png)
第20行是thread1对lock2进行加锁的位置.与我们上面分析的原因一致, 相互等待对方释放锁资源.

# 解决方案
既然知道原因了, 解决起来也就有方向了, 在本示例中, 只要让线程1和线程2不要同时执行就可以解决问题, 当然实际应用中死锁问题解决起来会复杂很多.
```java
        thread1.start();
	// 主线程等待thread1执行完再开始执行thread2, 即可避免资源竞争
        thread1.join();
        thread2.start();
```

![image.png](/images/2020/08/14/b6036e89-fe94-4b60-8d90-d6e18bc621bb.png)