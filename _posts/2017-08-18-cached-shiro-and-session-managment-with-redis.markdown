---
layout:     post
title:      "Cached Shiro and Session Managment with Redis"
subtitle:   "Redis !!!!!!!"
date:       2017-08-18
author:     "GeniusV"
header-img: "img/post-bg-2015.jpg"
catalog: false
tags:
    - Redis
    - Shiro
    - Spring MVC
    - Spring
---


#### Table of Content

<!-- MarkdownTOC -->

- [Shiro SecurityManager](#shiro-securitymanager)
- [SessionManager](#sessionmanager)
    - [SessionDao](#sessiondao)
- [CacheManager](#cachemanager)
    - [JedisShiroCache](#jedisshirocache)
- [JedisDao](#jedisdao)
- [Contact](#contact)

<!-- /MarkdownTOC -->


## Shiro SecurityManager

Today, let's see how to config Shiro Session and Cache in Web environment with Spring MVC. In this case, we use Shiro to store cache and session. Also, we can use other implementions, which are the same process.

The Demo is available [Here](https://github.com/GeniusV/Cache-Demo).

Here is is the SecurityManager configuration:

``` xml
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
    <property name="sessionManager" ref="sessionManager"/>
    <property name="cacheManager" ref="jedisShiroCacheManager"/>
    <property name="realm" ref="customRelam"/>
    <property name="rememberMeManager" ref="cookieRememberMeManager"/>
</bean>
```

We implement SessionManager and CacheManager to store them in Redis.

Here is the diagrame shows the dependencies:

![](/img/in-post/cached-shiro-and-session-managment-with-redis-securitymanager.png)

## SessionManager

Here is the SessionManager configuration:

``` xml
<bean id="sessionManager" class="org.apache.shiro.web.session.mgt.DefaultWebSessionManager">
    <property name="sessionValidationInterval" value="18000000"/>
    <property name="globalSessionTimeout" value="1800000"/>
    <property name="sessionDAO" ref="customShiroSessionDao"/>
    <property name="sessionListeners" ref="customSessionListener"/>
</bean>
```

- `sessionValidationInterval`: the interval in milliseconds that shiro checks the validation of sessions.
- `globalSessionTimeout`: sets the system-wide default time in milliseconds that any session may remain idle before expiring. 
- `sessionDAO`: the implemention of session operation.
- `sessionListeners`: Notification callback that occurs when the corresponding Session has stated, expired or stoped.

### SessionDao

To implement SessionDao, we have to implement `org.apache.shiro.session.mgt.eis.AbstractSessionDAO`. 

Here is the sessionDAO:
``` xml
<bean id="customShiroSessionDao" class="io.github.geniusv.shiro.session.CustomShiroSessionDao">
    <property name="shiroSessionRespository" ref="defaultShiroSessionRespository"/>
    <property name="sessionIdGenerator" ref="javaUuidSessionIdGenerator"/>
</bean>

 <bean id="javaUuidSessionIdGenerator" class="org.apache.shiro.session.mgt.eis.JavaUuidSessionIdGenerator"/>
```

I recommend not to directly inject jedisPool into this class, instead, I created an interface `ShiroSessionRespository` and an implemention `DefaultShiroSessionRespository` to perform operations. So that if we want to use other implementions to store sessions, we just need to implement `ShiroSessionRespository`.

- `sessionIdGenerator`: generate session id.
- `shiroSessionRespository`: custom shiroSessionRespository


By the way, I made a mistake here, the implemention of the `ShiroSessionRespository` should be `RedisShiroSessionRespository` or `DefaultRedisShiroSessionRespository`. But in this case, I only use Redis so it should be OK.

Here is is the implemention of `org.apache.shiro.session.mgt.eis.AbstractSessionDAO`:

``` java
public class CustomShiroSessionDao extends AbstractSessionDAO {

    private ShiroSessionRespository shiroSessionRespository;

    public ShiroSessionRespository getShiroSessionRespository() {
        return shiroSessionRespository;
    }

    public void setShiroSessionRespository(ShiroSessionRespository shiroSessionRespository) {
        this.shiroSessionRespository = shiroSessionRespository;
    }

    @Override

    protected Serializable doCreate(Session session) {
        Serializable sessionId = generateSessionId(session);
        assignSessionId(session, sessionId);
        getShiroSessionRespository().saveSession(session);
        return sessionId;
    }

    @Override
    protected Session doReadSession(Serializable serializable) {
        return shiroSessionRespository.getSession(serializable);
    }

    @Override
    public void update(Session session) throws UnknownSessionException {
        shiroSessionRespository.saveSession(session);
    }

    @Override
    public void delete(Session session) {
        shiroSessionRespository.deleteSession(session.getId());
    }

    @Override
    public Collection<Session> getActiveSessions() {
        return shiroSessionRespository.getAllSessions();
    }
}
```

#### ShiroSessionRespository

To achieve code reuse, we just catch exceptions, serialize and unserialize objects, add logs and convert type here, and use `JedisDao` to encapulate all bottom operations.

Here is the configuration:

``` xml
<bean id="defaultShiroSessionRespository"
      class="io.github.geniusv.shiro.session.impl.DefaultShiroSessionRespository">
    <property name="jedisDao" ref="jedisDao"/>
</bean>
```

Here is the interface and the implemention:

``` java
public interface ShiroSessionRespository {

    void saveSession(Session session);

    void deleteSession(Serializable sessionId);

    Session getSession(Serializable sessionId);

    Collection<Session> getAllSessions();
}

public class DefaultShiroSessionRespository implements ShiroSessionRespository {

    private JedisDao jedisDao;

    public JedisDao getJedisDao() {
        return jedisDao;
    }

    public void setJedisDao(JedisDao jedisDao) {
        this.jedisDao = jedisDao;
    }

    @Override
    public void saveSession(Session session) {
        if (session == null || session.getId() == null) {
            throw new NullPointerException("session is empty");
        }

        try {
            byte[] key = SerializeUtil.serialize(session.getId());
            byte[] value = SerializeUtil.serialize(session);
            Long sessionTimeOut = session.getTimeout() / 1000 + 60;
            jedisDao.saveValueByKey(key, value, sessionTimeOut.intValue());
        } catch (Exception e) {
            LoggerUtil.error(getClass(), e, "save session error, id:[%s]", session.getId());
        }
    }

    @Override
    public void deleteSession(Serializable sessionId) {
        if (sessionId == null) {
            throw new NullPointerException("session id is empty");
        }
        try {
            jedisDao.deleteByKey(SerializeUtil.serialize(sessionId));
        } catch (Exception e) {
            LoggerUtil.error(getClass(), e, "delete session throw exception: id:[%s]]", sessionId);
        }
    }

    @Override
    public Session getSession(Serializable sessionId) {
        if (sessionId == null) {
            throw new NullPointerException("session id is empty");
        }
        Session result = null;
        try {
            byte[] value = jedisDao.getValueByKey(SerializeUtil.serialize(sessionId));
            result = (Session) SerializeUtil.unserialize(value);
        } catch (Exception e) {
            LoggerUtil.error(getClass(), e, "get session throw exception: id:[%s]", sessionId);
        }
        return result;
    }

    @Override
    public Collection<Session> getAllSessions() {
        Collection<Session> result = null;
        try {
            Collection<byte[]> keys = jedisDao.getAllKeys("*".getBytes());
            for (byte[] item : keys) {
                result.add((Session) SerializeUtil.unserialize(item));
            }
        } catch (Exception e) {
            LoggerUtil.error(getClass(), e, "get all session throw exception");
        }
        return result;
    }
}
```

I injected the `JedisDao` in the class, for more infomation, see [`JedisDao`](#jedisdao).

## CacheManager
`CacheManager` manages cache, we have to implement `getCache` method, which means we need to new a cache. So we inject `JedisDao` into the `CacheManager` and pass it to the constructor of the cache.

Here is the configuration:

``` xml
<bean id="jedisShiroCacheManager" class="io.github.geniusv.shiro.cache.JedisShiroCacheManager">
    <property name="shiroJedisDao" ref="shiroJedisDao"/>
</bean>
```

Here is the implemention:

``` java
public class JedisShiroCacheManager implements CacheManager {

    private JedisDao jedisDao;

    public JedisDao getJedisDao() {
        return jedisDao;
    }

    public void setJedisDao(JedisDao jedisDao) {
        this.jedisDao = jedisDao;
    }

    @Override
    public <K, V> Cache<K, V> getCache(String s) throws CacheException {
        return new JedisShiroCache<>(jedisDao);
    }
}
```

See [`JedisDao`](#jedisdao) for more infomation about it.

### JedisShiroCache

`JedisShiroCache` implements `org.apache.shiro.cache.Cache`, we can catch exceptions, do logs, and some other things here, all bottom operations will be given to `JedisDao`.

Here it is:

``` java
public class JedisShiroCache<k, v> implements Cache<k, v> {

    private ShiroJedisDao dao;

    public JedisShiroCache(ShiroJedisDao dao) {
        this.dao = dao;
    }

    @Override
    public v get(k key) throws CacheException {
        byte[] byteKey = SerializeUtil.serialize(key);
        byte[] byteValue = new byte[0];
        try {
            byteValue = dao.getValueByKey(byteKey);
        } catch (Exception e) {
            LoggerUtil.error(getClass(), "get shiro cache value by cache throw exception", e);
        }
        v result = (v) SerializeUtil.unserialize(byteValue);
        if (result != null) {
            LoggerUtil.debug(getClass(), "shiro getting cache: getAllKeys: %s, value: %s", key.toString(), result.toString());
        }
        return result;
    }

    @Override
    public v put(k key, v value) throws CacheException {
        LoggerUtil.debug(getClass(), "shiro putting cache: getAllKeys: %s, value: %s", key.toString(), value.toString());
        v previous = get(key);
        try {
            dao.saveValueByKey(SerializeUtil.serialize(key), SerializeUtil.serialize(value), -1);
        } catch (Exception e) {
            LoggerUtil.error(getClass(), "put shiro cache throw exception", e);
        }
        return previous;
    }

    @Override
    public v remove(k key) throws CacheException {
        LoggerUtil.debug(getClass(), "shiro deleting cache: getAllKeys: %s", key.toString());
        v previous = get(key);
        try {
            dao.deleteByKey(SerializeUtil.serialize(key));
        } catch (Exception e) {
            LoggerUtil.error(getClass(), "remove shiro cache throw exception", e);
        }
        return previous;
    }

    @Override
    public void clear() throws CacheException {
        LoggerUtil.debug(getClass(), "shiro clearing all cache...");
        try {
            dao.flushdb();
        } catch (Exception e) {
            LoggerUtil.error(getClass(), "clear shiro cache throw exception", e);
        }
    }

    @Override
    public int size() {
        if (keys() == null)
            return 0;
        return keys().size();
    }

    @Override
    public Set<k> keys() {
        return null;
    }

    @Override
    public Collection<v> values() {
        return null;
    }
}

```

## JedisDao

`JedisDao` encapulates some bottom operations, we need to inject the `jedispool` and the `dbindex`. `dbindex` is an option. if we don't config `dbindex`, all operations will be performed on redis database `0`, which is the default redis database.

here is the `JedisDao` configuration:

``` xml
<bean id="JedisDao" class="io.github.geniusv.jedis.JedisDao">
    <property name="jedispool" ref="jedispool"/>
    <property name="dbindex" value="1"/>
</bean>
```

here is the `JedisDao`:

``` java
public class JedisDao {

    private int dbindex = 0;

    private jedispool jedispool;

    public jedispool getjedispool() {
        return jedispool;
    }

    public void setjedispool(jedispool jedispool) {
        this.jedispool = jedispool;
    }

    public int getdbindex() {
        return dbindex;
    }

    public void setdbindex(int dbindex) {
        this.dbindex = dbindex;
    }

    public jedis getjedis() {
        jedis jedis = null;
        try {
            jedis = jedispool.getresource();
            jedis.select(dbindex);
        } catch (jedisconnectionexception e) {
            loggerutil.error(getclass(), "get redis error", e);
        }

        return jedis;
    }

    public void returnresource(jedis jedis, boolean isbroken) {
        if (jedis == null)
            return;
        jedis.close();
    }

    public byte[] getvaluebykey(byte[] key) throws exception {
        jedis jedis = null;
        byte[] result = null;
        boolean isbroken = false;
        try {
            jedis = getjedis();
            result = jedis.get(key);
        } catch (exception e) {
            isbroken = true;
            throw e;
        } finally {
            returnresource(jedis, isbroken);
        }
        return result;
    }

    public void deletebykey(byte[] key) throws exception {
        jedis jedis = null;
        boolean isbroken = false;
        try {
            jedis = getjedis();
            jedis.del(key);
        } catch (exception e) {
            isbroken = true;
            throw e;
        } finally {
            returnresource(jedis, isbroken);
        }
    }

    public void savevaluebykey(byte[] key, byte[] value, int expiretime)
            throws exception {
        jedis jedis = null;
        boolean isbroken = false;
        try {
            jedis = getjedis();
            jedis.set(key, value);
            if (expiretime > 0)
                jedis.expire(key, expiretime);
        } catch (exception e) {
            isbroken = true;
            throw e;
        } finally {
            returnresource(jedis, isbroken);
        }
    }

    public void flushdb() throws exception {
        jedis jedis = null;
        boolean isbroken = false;
        try {
            jedis = getjedis();
            jedis.flushdb();
        } catch (exception e) {
            isbroken = true;
            throw e;
        } finally {
            returnresource(jedis, isbroken);
        }
    }

    public collection<byte[]> getallkeys(byte[] pattern) {
        jedis jedis = null;
        set<byte[]> result = null;
        boolean isbroken = false;
        try {
            jedis = getjedis();
            result = jedis.keys(pattern);
        } catch (exception e) {
            isbroken = true;
            throw e;
        } finally {
            returnresource(jedis, isbroken);
        }
        return result;
    }
}
```

we can make it as an abstract class, then we can custom different type of `JedisDao` in different scenarios.

## Contact

Email: eliot310100@outlook.com
