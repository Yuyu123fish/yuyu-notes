# 线程

## 创建线程的方式有哪些

- 继承`Thread`，重写`run`方法，调用`start()`，这里`start()`只能调用一次
- 实现`Runnable`接口，并实现`run`方法，这里不能抛异常，且没有返回值

- 如果是直接调用它的run方法，那么是当前线程执行，可以有无数次
- 如果是交给`Thread`执行，通过`start()`执行，只能执行一次

- 实现`Callable`接口，并实现`call`方法，这里可以抛异常，同时返回一个泛型返回值。需要配合`FutureTask<> ft`来使用，然后将`ft`放到`Thread`里面，调用`start`方法即可。结束后使用`ft.get()`来拿到对应的泛型结果
- 使用线程池来创建线程（项目中常用）。

_线程的`run()`和`start()`有什么区别？_

- `start()`用来启动线程，通过该线程调用`run()`方法中所定义的逻辑代码。`start()`方法只能被调用一次。
- `run()`封装了要执行的代码，可以被调用多次。

## 线程包含哪些状态，状态之间如何变化
![](assets/Java线程状态.png)

线程包含：NEW 新建，RUNNABLE 可运行，TERMINATED 死亡，BLOCKED 阻塞，WAITING 等待，TIMED_WAITING 计时等待。一共六个状态。

## `notify()`和`notifyAll()`有什么区别

- `notify()`是随机唤醒一个线程。
- `notifyAll()`是唤醒全部线程。

## `wait()`,`wait(long)`,`sleep(long)`有什么区别

这三个方法的效果就是让当前线程暂时放弃CPU的使用权，进入阻塞状态。

它们的核心区别有三点：

- 所属类不同

- `sleep(long)`属于`Thread`的静态方法
- `wait()`和`wait(long)`属于`Obejct`的成员方法，每个对象都有

- 醒来时机不同

- 执行`sleep(long)`和`wait(long)`的线程都会在等待相应毫秒后醒来
- `wait()`和`wait(long)`还可以被`notify`唤醒，`wait()`如果不被唤醒会一直等待下去。
- 它们都可以被打断唤醒

- 锁特性不同（重点）

- `wait`方法的调用必须先获取`wait`对象的锁，而`sleep`则无此限制
- `wait`方法**执行后会主动释放对象锁**，允许其他线程获得该对象锁。
- `sleep`方法如果在`synchronized`代码块中执行，**并不会释放对象锁**。

## 可以直接调用Thread类的run方法吗？

可以直接调用，但是`run()`方法是让调用者线程去执行，这就不是多线程了，而如果是调用`start()`来让线程处于就绪状态然后执行，是属于多线程的。

## 如何停止一个正在运行的线程

- 使用退出标志，让线程执行完，使线程正常退出，也就是当`run`方法完成后线程终止。
- 使用`stop`强行终止（已停止使用，作废）。
- 使用`interrupt`方法中断线程

- 打断阻塞的线程(`sleep,wait,join`)的线程，会抛出`InterruptedException`
- 打断正常的线程，可以根据打断状态来标记是否退出线程

# 多线程

# 新建三个线程，如何保证顺序执行

**使用**`t1.join()`**来解决**

`t1.join()`的作用是，阻塞当前线程，只有等到`t1`线程执行结束后，才开始执行。

**使用**`synchronized`**来解决**

```java
package study;

public class ThreeThreadPrint0to100 {
    private static final Object LOCK = new Object();
    private static volatile int count = 0;
    private static final int MAX = 100;

    public static void main(String[] args) {
        Thread thread0 = new Thread(new MyRunnable(0));
        Thread thread1 = new Thread(new MyRunnable(1));
        Thread thread2 = new Thread(new MyRunnable(2));
        thread0.start();
        thread1.start();
        thread2.start();
    }

    static class MyRunnable implements Runnable{
        private final int seq;
        public MyRunnable(int seq){
            this.seq = seq;
        }
        @Override
        public void run() {
            while(count <= MAX){
                synchronized (LOCK){
                    while(count % 3 != seq){
                        try {
                            LOCK.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    if(count > MAX){
                        LOCK.notifyAll();
                        return;
                    }
                    System.out.println("seq:" + seq + " print " + count);
                    count++;
                    LOCK.notifyAll();//防止唤醒一个继续wait的造成死锁
                }
            }
        }
    }
}
```

**使用**`Semaphore`**来解决**

```java
package study;

import java.util.concurrent.Semaphore;

public class ThreeThreadPrintABC {
    private final Semaphore sem1 = new Semaphore(1);
    private final Semaphore sem2 = new Semaphore(0);
    private final Semaphore sem3 = new Semaphore(0);
    private static final int round = 3;

    public static void main(String[] args) {
        ThreeThreadPrintABC printer = new ThreeThreadPrintABC();
        new Thread(() -> print('A', printer.sem1, printer.sem2)).start();
        new Thread(() -> print('B', printer.sem2, printer.sem3)).start();
        new Thread(() -> print('C', printer.sem3, printer.sem1)).start();
    }
    private static void print(char c, Semaphore cur, Semaphore next){
        try{
            for (int i = 0; i < round; i++) {
                cur.acquire();
                System.out.println(Thread.currentThread().getName() + ":" + c);
                next.release();
            }
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```