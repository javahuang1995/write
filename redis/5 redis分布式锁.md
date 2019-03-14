```java
String script = 
"if redis.call('get', KEYS[1]) == ARGV[1] 
then return redis.call('del', KEYS[1]) 
    else return 0 
        end";
```



```java
package com.huang.redis.util;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;

import java.util.Collections;
import java.util.concurrent.TimeUnit;

/**
 * @Auther: 宁哥
 * @Date: 2019/3/13 19:45
 * @Description:
 */
@Component
public class MyLock
{
    @Autowired
    private JedisPool jedisPool;

    private static final int DEFAULT_SLEEP_TIME = 10;
    private static final String LOCK_MSG = "1";
    private static final String UNLOCK_MSG = "0";

    private static final String LOCK_PREFIX = "PREFIX";
    private static final String SET_IF_NOT_EXIST = "NX";
    private static final String SET_WITH_EXPIRE_TIME = "10";
    private static final int DEFAULT_EXPIRE_TIME = 10;


    /**
     * value应该传入当前的线程ID当做value,并在删除之前验证key对应的value是不是自己线程的ID。
     * 这里有个问题，就是A拿到锁之后，设置了超时时长是SET_WITH_EXPIRE_TIME，也就是缓存数据的存活时间。
     * 那么问题来了，如果A业务还没有执行完，锁(缓存数据)已经被删掉了怎么办？？？？
     */
    public boolean lock(String key, String value, int timeout) throws InterruptedException
    {
        Jedis jedis = jedisPool.getResource();

        while (timeout >= 0)
        {
            String result = jedis.set(LOCK_PREFIX + key, value, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, DEFAULT_EXPIRE_TIME);
            if (LOCK_MSG.equals(result))
            {
                jedis.close();
                return true;
            }
            TimeUnit.SECONDS.sleep(timeout);
            timeout = timeout - DEFAULT_SLEEP_TIME;
        }
        return false;
    }

    /**
     * 因为判断equal和del是2个独立操作，不是原子性，所以这里用到lua脚本，lua脚本在redis里面是原子性的。
     * if（threadId .equals(redisClient.get(key))）
     * {
     *     del(key)
     * }
     * 这里value传入线程id即可。
     * 这里需要比较Redis里面的key和value是否一致
     */
    public boolean unlock(String key, String value)
    {
        Jedis jedis = jedisPool.getResource();

        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
        Object result = jedis.eval(script, Collections.singletonList(LOCK_PREFIX + key), Collections.singletonList(value));

        jedis.close();
        return UNLOCK_MSG.equals(result);
    }

}

//Collections.singletonList
//这个方法主要用于只有一个元素的优化，减少内存分配，无需分配额外的内存，可以从SingletonList内部类看得出来,由于只有一个element,因此可以做到内存分配最小化，相比之下ArrayList的DEFAULT_CAPACITY=10个。


```

我们可以让获得锁的线程开启一个**守护线程**，用来给快要过期的锁“续航”。

如果说锁的过期时间是30秒，当过去了29秒，线程A还没执行完，这时候守护线程会执行expire指令，为这把锁“续命”30秒。守护线程从第29秒开始执行，每20秒执行一次。

![](..\images\搜狗截图20190313202832.png)

我为xx续一秒。。。守护线程真是好用啊。。

当线程A执行完任务，会显式关掉守护线程。

另一种情况，如果节点1 忽然断电，由于线程A和守护线程在同一个进程，守护线程也会停下。这把锁到了超时的时候，没人给它续命，也就自动释放了。

![](..\images\搜狗截图20190313203100.png)