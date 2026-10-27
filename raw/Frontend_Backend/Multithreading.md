## 1. 进程和线程
- 进程：应用程序的执行实例，有独立内存空间和系统资源。
- 线程：CPU 调度和分派的基本单位，进程中执行运算的最小单位，可完成独立顺序控制流程。
- 多线程：一个进程中同时运行多个线程，完成不同工作。
- 多个线程交替占用 CPU 资源，而非真正并行执行。
- 好处：充分利用 CPU 资源、简化编程模型、良好用户体验。

## 2. 主线程
- Java 提供 `java.lang.Thread` 类支持多线程编程。
- `main()` 方法为主线程入口。
- 主线程必须最后完成执行，因为它执行各种关闭动作。
```java
Thread t = Thread.currentThread();
System.out.println("当前线程是:" + t.getName());
```

## 3. 创建线程
### 3.1 继承 Thread 类
1. 定义 MyThread 类继承 Thread。
2. 重写 `run()` 方法，编写线程执行体。
3. 创建线程对象，调用 `start()` 启动。
```java
public class MyThread extends Thread {
    public void run() {
        for(int i=1;i<100;i++){
            System.out.println(Thread.currentThread().getName()+":"+i);
        }
    }
}
MyThread thread = new MyThread();
thread.start();
```
- 多个线程交替执行，不是真正并行；每次执行时长由 CPU 时间片决定。
- `run()` 与 `start()` 区别：
  - 直接调用 `run()`：只有主线程一条执行路径，相当于普通方法调用。
  - 调用 `start()`：启动新线程，主线程和子线程并行交替执行。

### 3.2 实现 Runnable 接口
1. 定义 MyRunnable 类实现 Runnable 接口。
2. 实现 `run()` 方法。
3. 创建线程对象，调用 `start()`。
```java
public class MyRunnable implements Runnable {
    public void run() {
        for(int i=1;i<100;i++){
            System.out.println(Thread.currentThread().getName()+":"+i);
        }
    }
}
MyRunnable myRunnable = new MyRunnable();
Thread thread = new Thread(myRunnable);
thread.start();
```
- 比较：
  - 继承 Thread：编写简单，可直接操作线程，适用于单继承。
  - 实现 Runnable：避免单继承局限性，便于共享资源。

## 4. 线程状态
- 创建状态
- 就绪状态
- 运行状态
- 阻塞状态
- 死亡状态

## 5. 线程调度
- 按照特定机制为多个线程分配 CPU 使用权。
- 常用方法：
  - `void setPriority(int newPriority)`：更改优先级。
  - `static void sleep(long millis)`：休眠指定毫秒。
  - `void join()`：等待该线程终止。
  - `static void yield()`：暂停当前线程，允许其他同优先级线程获得运行机会。
  - `void interrupt()`：中断线程。
  - `boolean isAlive()`：测试线程是否活动。

### 5.1 线程优先级
- 1~10，1 最低，默认 5。
- 优先级高的线程获得 CPU 资源概率较大。
```java
t1.setPriority(Thread.MAX_PRIORITY);
t2.setPriority(Thread.MIN_PRIORITY);
```

### 5.2 sleep
- 让线程暂时睡眠指定时长，进入阻塞状态；睡眠时间过后再进入可运行状态。
```java
Thread.sleep(1000); // 休眠1秒
```

### 5.3 join
- 使当前线程暂停执行，等待其他线程结束后再继续执行本线程。
```java
temp.join();
```
- 需处理 InterruptedException。

### 5.4 yield
- 暂停当前线程，允许其他具有相同优先级的线程获得运行机会。
- 该线程处于就绪状态，不转为阻塞状态。

## 6. 多线程共享数据问题
- 示例：网络购票。
- 问题：
  - 不是从第1张票开始。
  - 多人抢到同一张票。
  - 有些票号没有被抢到。
- 原因：多线程并发修改共享数据，非原子操作。

## 7. 线程同步
### 7.1 同步方法
```java
访问修饰符 synchronized 返回类型 方法名(参数列表) { ... }
// 或
synchronized 访问修饰符 返回类型 方法名(参数列表) { ... }
```
示例：
```java
public synchronized void sale() {
    if (count <= 0) { flag = true; return; }
    // 修改数据、显示信息
}
```

### 7.2 同步代码块
```java
synchronized(syncObject) {
    // 需要同步的代码
}
```
- syncObject 通常为 this。
- 多个并发线程访问同一资源的同步代码块时：
  - 同一时刻只能有一个线程进入 synchronized(this) 同步代码块。
  - 一个线程访问 synchronized(this) 同步代码块时，其他 synchronized(this) 同步代码块同样被锁定。
  - 一个线程访问 synchronized(this) 同步代码块时，其他线程可以访问该资源的非 synchronized(this) 同步代码。

## 8. 线程安全的类型
- ArrayList 的 `add()` 方法为非同步方法，多线程添加可能出现数据不一致；ArrayList 非线程安全。
- 线程安全：同步，效率低，适合多线程并发共享资源。
- 非线程安全：不同步，效率高，适合单线程。
- 常见对比：
  - Hashtable：线程安全，效率较低，键和值都不允许 null。
  - HashMap：非线程安全，效率较高，键和值都允许 null。
  - StringBuffer：线程安全。
  - StringBuilder：非线程安全。

## 9. 并发包
- `java.util.concurrent` 提供方便并发编程的类：
  - 阻塞队列 BlockingQueue
  - 延迟队列 DelayQueue
  - 闭锁 CountDownLatch
  - 线程池 ExecutorService
- 第三种创建线程方式：`Callable`，能够获取线程执行结果。

## 10. 练习
- 模拟多人爬山。
- 线程优先级。
- 模拟叫号看病。
- 模拟接力赛跑。
- 网络购票：桃跑跑、张票票、黄牛党共同抢10张票，黄牛党只能抢一张。

## 11. 小结
- 线程和进程区别。
- 创建线程两种方式：继承 Thread、实现 Runnable。
- 线程调度方法：优先级、sleep、join、yield。
- 线程同步：synchronized 方法/代码块。
- 线程安全类型与非线程安全类型。