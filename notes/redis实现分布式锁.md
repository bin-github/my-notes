#### 1、用redis来实现分布式锁需要注意的问题
>1、加锁时必须保证原子性

>2、谁加的锁谁释放，而且只能释放自己加的锁

>3、锁应该有过期时间，以防线程/进程出现锁饥饿现象

>4、下面是一个示例：使用jedis实现，客户端使用统一的redis服务的毫秒数来标记


```
private int LOCK_TIMEOUT = 5 * 1000;

/**
     * 获取锁
     * @param lockKey
     * @return
     */
    public long lock(String lockKey) {
        log.info("key={}开始加锁==========",lockKey);
        long lockTime = getLockTime();
        String value = (String.valueOf(lockTime));
        // 该语句在redis中保证了原子性
        boolean isLock = jedisService.set(lockKey, value,"nx","px",LOCK_TIMEOUT);
        if (isLock) {
            log.info("key={}加锁成功=======1超时时间={}",lockKey,LOCK_TIMEOUT);
            return lockTime;
        } else {
            Long currtLockTime = Long.parseLong(jedisService.get(lockKey));
            if (currtLockTime != null && currtLockTime < System.currentTimeMillis()) {
                // 判断是否为空，不为空的情况下，说明已经失效，如果被其他线程设置了值，则第二个条件判断是无法执行
                Long oldLockTimeout = Long.valueOf(jedisService.getAndSet(lockKey, value));
                // 获取上一个锁到期时间，并设置现在的锁到期时间
                if (oldLockTimeout != null && oldLockTimeout.equals(currtLockTime)) {
                    // 如过这个时候，多个线程恰好都到了这里，但是只有一个线程的设置值和当前值相同，他才有权利获取锁
                    log.info("key={}加锁成功=======2",lockKey);
                    jedisService.expire(lockKey, LOCK_TIMEOUT); 
                    return lockTime;
                }
            }
        }
        return 0L;
    }
    
    
     /**
     * 解锁
     * @param lockKey
     * @param lockTime
     * @return
     */
    public void unLock(String lockKey,long lockTime) {
        log.info("key={}开始解锁==========",lockKey);
        String oldTime = jedisService.get(lockKey);
        if (StringUtils.isBlank(oldTime)){
            log.warn("key={}锁不存在--------",lockKey);
            return;
        }
        Long currtLockTime = Long.parseLong(oldTime);
        if (currtLockTime != null && currtLockTime == lockTime) {
            jedisService.del(lockKey);
            log.info("key={}解锁成功---------",lockKey);
        }else {
            log.warn("key={}锁已过期或该锁不存在,加锁时间={}--------",lockKey,currtLockTime);
        }
    }
    
    /**
     * 获取redis服务时间
     * @return
     */
    private long getLockTime(){
        long time = jedisService.getRedisTime();
        if (time == 0){
            time = System.currentTimeMillis() + LOCK_TIMEOUT;
        }
        return time;
    }
```
