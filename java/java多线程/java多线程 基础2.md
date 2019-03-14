## ReentrantLock

可重入锁





## 同步容器1

使用wait/notify

```java
/**
 * 面试题：写一个固定容量同步容器，拥有put和get方法，以及getCount方法，
 * 能够支持2个生产者线程以及10个消费者线程的阻塞调用
 * <p>
 * 使用wait和notify/notifyAll来实现
 *
 * @author mashibing
 */
package c_021;

import java.util.LinkedList;
import java.util.concurrent.TimeUnit;

public class MyContainer1<T>
{
    //这个List就是共享资源
    //所以锁要使用在和共享资源一个地方，互斥方才都可以访问。synchronized就是this.lock();
    final private LinkedList<T> lists = new LinkedList<>();
    final private int MAX = 10; //最多10个元素
    private int count = 0;


    public synchronized void put(T t)
    {
        while (lists.size() == MAX)
        { //想想为什么用while而不是用if？
            try
            {
                this.wait(); //effective java
            }
            catch (InterruptedException e)
            {
                e.printStackTrace();
            }
        }

        lists.add(t);
        ++count;
        this.notifyAll(); //通知消费者线程进行消费
    }

    public synchronized T get()
    {
        T t = null;
        while (lists.size() == 0)
        {
            try
            {
                this.wait();
            }
            catch (InterruptedException e)
            {
                e.printStackTrace();
            }
        }
        t = lists.removeFirst();
        count--;
        this.notifyAll(); //通知生产者进行生产
        return t;
    }

    public static void main(String[] args)
    {
        MyContainer1<String> c = new MyContainer1<>();
        //启动消费者线程
        for (int i = 0; i < 10; i++)
        {
            new Thread(() ->
            {
                for (int j = 0; j < 5; j++) System.out.println(c.get());
            }, "c" + i).start();
        }

        try
        {
            TimeUnit.SECONDS.sleep(2);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }

        //启动生产者线程
        for (int i = 0; i < 2; i++)
        {
            new Thread(() ->
            {
                for (int j = 0; j < 25; j++) c.put(Thread.currentThread().getName() + " " + j);
            }, "p" + i).start();
        }
    }
}

```



## 同步容器2

如果用手动锁Lock,那么wait/notify肯定不能用了，

所以这里用Condition

```java
/**
 * 面试题：写一个固定容量同步容器，拥有put和get方法，以及getCount方法，
 * 能够支持2个生产者线程以及10个消费者线程的阻塞调用
 * 
 * 使用wait和notify/notifyAll来实现
 * 
 * 使用Lock和Condition来实现
 * 对比两种方式，Condition的方式可以更加精确的指定哪些线程被唤醒
 * 
 * @author mashibing
 */
package c_021;

import java.util.LinkedList;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class MyContainer2<T> {
	final private LinkedList<T> lists = new LinkedList<>();
	final private int MAX = 10; //最多10个元素
	private int count = 0;
	
	private Lock lock = new ReentrantLock();
	private Condition producer = lock.newCondition();
	private Condition consumer = lock.newCondition();
	
	public void put(T t) {
		try {
			lock.lock();
			while(lists.size() == MAX) { //想想为什么用while而不是用if？
				producer.await();
			}
			
			lists.add(t);
			++count;
			consumer.signalAll(); //通知消费者线程进行消费
		} catch (InterruptedException e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}
	
	public T get() {
		T t = null;
		try {
			lock.lock();
			while(lists.size() == 0) {
				consumer.await();
			}
			t = lists.removeFirst();
			count --;
			producer.signalAll(); //通知生产者进行生产
		} catch (InterruptedException e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
		return t;
	}
	
	public static void main(String[] args) {
		MyContainer2<String> c = new MyContainer2<>();
		//启动消费者线程
		for(int i=0; i<10; i++) {
			new Thread(()->{
				for(int j=0; j<5; j++) System.out.println(c.get());
			}, "c" + i).start();
		}
		
		try {
			TimeUnit.SECONDS.sleep(2);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		
		//启动生产者线程
		for(int i=0; i<2; i++) {
			new Thread(()->{
				for(int j=0; j<25; j++) c.put(Thread.currentThread().getName() + " " + j);
			}, "p" + i).start();
		}
	}
}

```

