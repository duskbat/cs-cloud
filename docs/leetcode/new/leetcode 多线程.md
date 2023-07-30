# leetcode 多线程

## 1279. 红绿灯路口 easy

> 对共享资源的操作 加锁

**解法: Lock**

```java
public class TrafficLight {
    private final Lock lock = new ReentrantLock();
    private volatile int road = 1;

    public void carArrived(
            int carId,           // ID of the car
            int roadId,          // ID of the road the car travels on. Can be 1 (road A) or 2 (road B)
            int direction,       // Direction of the car
            Runnable turnGreen,  // Use turnGreen.run() to turn light to green on current road
            Runnable crossCar    // Use crossCar.run() to make car cross the intersection
    ) {
        lock.lock();
        if (road != roadId) {
            turnGreen.run();
            road = roadId;
        }
        crossCar.run();
        lock.unlock();
    }
}
```

**解法: CAS**

```java
public class TrafficLight {
    private final AtomicInteger atomic = new AtomicInteger();
    private volatile int road = 1;


    public void carArrived(
            int carId,           // ID of the car
            int roadId,          // ID of the road the car travels on. Can be 1 (road A) or 2 (road B)
            int direction,       // Direction of the car
            Runnable turnGreen,  // Use turnGreen.run() to turn light to green on current road
            Runnable crossCar    // Use crossCar.run() to make car cross the intersection
    ) {
        try {
            while (!atomic.compareAndSet(0, 1)) {
                Thread.sleep(1);
            }
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        if (road != roadId) {
            turnGreen.run();
            road = roadId;
        }
        crossCar.run();
        try {
            while (!atomic.compareAndSet(1, 0)) {
                Thread.sleep(1);
            }
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

**解法: Synchronized**

```java
public class TrafficLight {
    private final Object LOCK = new Object();
    private volatile int road = 1;

    public void carArrived(
            int carId,           // ID of the car
            int roadId,          // ID of the road the car travels on. Can be 1 (road A) or 2 (road B)
            int direction,       // Direction of the car
            Runnable turnGreen,  // Use turnGreen.run() to turn light to green on current road
            Runnable crossCar    // Use crossCar.run() to make car cross the intersection
    ) {
        synchronized (LOCK) {
            if (road != roadId) {
                turnGreen.run();
                road = roadId;
            }
            crossCar.run();
        }
    }
}
```

**解法: Semaphore**

```java
public class TrafficLight {
    Semaphore semaphore = new Semaphore(1);
    private volatile int road = 1;

    public void carArrived(
            int carId,           // ID of the car
            int roadId,          // ID of the road the car travels on. Can be 1 (road A) or 2 (road B)
            int direction,       // Direction of the car
            Runnable turnGreen,  // Use turnGreen.run() to turn light to green on current road
            Runnable crossCar    // Use crossCar.run() to make car cross the intersection
    ) {
        try {
            semaphore.acquire();
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        if (road != roadId) {
            turnGreen.run();
            road = roadId;
        }
        crossCar.run();
        semaphore.release();
    }
}
```

## 1188. 设计有限阻塞队列 mid

实现一个拥有如下方法的线程安全有限阻塞队列：

- BoundedBlockingQueue(int capacity)构造方法初始化队列，其中capacity代表队列长度上限。
- void enqueue(int element)在队首增加一个element. 如果队列满，调用线程被阻塞直到队列非满。
- int dequeue()返回队尾元素并从队列中将其删除. 如果队列为空，调用线程被阻塞直到队列非空。
- int size()返回当前队列元素个数。
- 你的实现将会被多线程同时访问进行测试。每一个线程要么是一个只调用enqueue方法的生产者线程，要么是一个只调用dequeue方法的消费者线程。size方法将会在每一个测试用例之后进行调用。

请不要使用内置的有限阻塞队列实现，否则面试将不会通过。

> Condition 控制锁

```java
public class BoundedBlockingQueue {

    private final AtomicInteger size = new AtomicInteger(0);
    private final int capacity;
    private final Queue<Integer> container = new LinkedList<>();
    private final ReentrantLock lock = new ReentrantLock();
    //用来通知生产（入队）线程等待await还是可以执行signal
    private final Condition producer = lock.newCondition();
    //用来通知消费（出队）线程等待await还是可以执行signal
    private final Condition consumer = lock.newCondition();

    public BoundedBlockingQueue(int capacity) {
        this.capacity = capacity;
    }

    public void enqueue(int element) throws InterruptedException {
        lock.lock();
        try {
            while (size.get() >= capacity) {
                producer.await();
            }
            container.add(element);
            size.incrementAndGet();

            consumer.signal();
        } finally {
            lock.unlock();
        }
    }

    public int dequeue() throws InterruptedException {
        lock.lock();
        try {
            while (size.get() == 0) {
                consumer.await();
            }
            int lastValue = container.remove();
            size.decrementAndGet();

            producer.signal();
            return lastValue;
        } finally {
            lock.unlock();
        }
    }

    public int size() {
        lock.lock();
        try {
            return size.get();
        } finally {
            lock.unlock();
        }
    }
}
```

## 1242. 多线程网页爬虫 mid

题目描述叨逼叨, 实际就是让多线程请求资源

> 注意下 thread.join() 的使用方式

```java
public class Solution {

    Set<String> visited = new HashSet<>();
    String rootHostname;
    HtmlParser htmlParser;

    class Task extends Thread {
        String url;

        public Task(String url) {
            this.url = url;
        }

        @Override
        public void run() {
            List<String> subUrls = htmlParser.getUrls(url);
            // 持有线程的引用
            List<Thread> subTasks = new ArrayList<>();
            for (String s : subUrls) {
                if (visited.contains(s)
                // 必须域名一致的路径
                    || !rootHostname.equals(getHostName(s))) continue;
                addUrl(visited, s);
                Thread task = new Task(s);
                subTasks.add(task);
                task.start();
            }
            for (Thread task : subTasks) {
                try {
                    task.join();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }

    public List<String> crawl(String startUrl, HtmlParser htmlParser) {
        this.htmlParser = htmlParser;
        this.rootHostname = getHostName(startUrl);
        addUrl(visited, startUrl);
        Thread thread = new Task(startUrl);
        thread.start();
        try {
            thread.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return new ArrayList<>(visited);
    }

    static String getHostName(String url) {
        final int start = 7;
        int end = url.indexOf('/', start);
        if (end == -1) {
            end = url.length();
        }
        return url.substring(start, end);
    }

    static synchronized void addUrl(Set<String> result, String url) {
        result.add(url);
    }
}
```

### 1115. 交替打印 FooBar

两个不同的线程将会共用一个 FooBar 实例：

- 线程 A 将会调用 foo()方法，而
- 线程 B 将会调用 bar()方法

请设计修改程序，以确保 "foobar" 被输出 n 次。

```java
class FooBar {
    private int n;
    private boolean flag = true;
    private ReentrantLock lock = new ReentrantLock();
    private Condition condition1 = lock.newCondition();
    private Condition condition2 = lock.newCondition();

    public FooBar(int n) {
        this.n = n;
    }

    public void foo(Runnable printFoo) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            try {
                lock.lock();
                while (!flag) condition1.await();
                printFoo.run();
                flag = false;
                condition2.signal();
            } finally {
                lock.unlock();
            }
        }
    }

    public void bar(Runnable printBar) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            try {
                lock.lock();
                while (flag) condition2.await();
                printBar.run();
                condition1.signal();
            } finally {
                lock.unlock();
            }
        }
    }
}
```

### 三个线程 循环打印 123

```java
class Solution {
    static int cnt = 1;
    static ReentrantLock lock = new ReentrantLock();
    static Condition condition1;
    static Condition condition2;
    static Condition condition3;


    public static void main(String[] args) {
        condition1 = lock.newCondition();
        condition2 = lock.newCondition();
        condition3 = lock.newCondition();
        Worker worker1 = new Worker(1, condition1, condition2);
        Worker worker2 = new Worker(2, condition2, condition3);
        Worker worker3 = new Worker(0, condition3, condition1);
        worker1.start();
        worker2.start();
        worker3.start();
    }

    static class Worker extends Thread {
        public Worker(int idx, Condition condition, Condition nextCondition) {
            this.idx = idx;
            this.condition = condition;
            this.nextCondition = nextCondition;
        }

        int idx;
        Condition condition;
        Condition nextCondition;

        @Override
        public void run() {
            Worker worker = (Worker) Thread.currentThread();
            while (true) {
                // 非公平锁会在这里抢锁
                lock.lock();
                try {
                    // 释放锁
                    if (cnt % 3 != worker.idx) worker.condition.await();
                    if (cnt < 100) System.out.println(worker + ":" + cnt++);
                    // 通知别人抢锁
                    worker.nextCondition.signal();
                    if (cnt >= 100) break;
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    lock.unlock();
                }
            }
        }
    }
}
```