---
layout:     post
title:      "Spring Transaction"
subtitle:   ""
date:       2017-01-19
author:     "GeniusV"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Spring Framework
    - Java
    - JDBC
    - SQL
---


## Transaction
- Transaction 的属性：  
    - propagation   
    指定事务的传播行为，即当前的事务方法被另外一个事务方法调用时如何使用事务， 默认取值为 `REQUIRED` ， 即使用调用方法的事务。
	- isolation  
  	指定事务的隔离级别，最常用的取值为 `READ_COMMITTED`.
  	- rollback 属性  
	默认Spring 的声明式事务对所有的运行时异常进行回滚，也可以通过对赢的属性进行设置。 
    - read-only   
    指定事务是否只为只读，只读事务运行速度更快。
    - timeout 
    单位为秒，指定强制回滚之前事务可以占用的时间
- 配置方式
    - Annotation： 配置`tx:annotation-driven transaction-manager`   

#### Demo
接口：

``` java
public interface BookShopDao {
    public int findBookPriceByID(String id);
    public void updateBookStock(String id);
    public void updateUserAccount(String userName, int price);
}
```

接口实现类：

``` java
@Repository("bookShopDao")
public class BookShopDaoImpl implements BookShopDao {
    @Autowired
    private JdbcTemplate jdbcTemplate;
    @Override
    public int findBookPriceByID(String id) {
        String sql = "select price from book where id = ?";
        return jdbcTemplate.queryForObject(sql, Integer.class, id);
    }
    @Override
    public void updateBookStock(String id) {
        String sql2 = "select stock from book_stock where id =?";
        int stock = jdbcTemplate.queryForObject(sql2, Integer.class, id);
        if (stock == 0) {
            throw new BookStockException("no stock");
        }
        String sql = "update book_stock set stock = stock - 1 where id =?";
        jdbcTemplate.update(sql, id);
    }
    @Override
    public void updateUserAccount(String userName, int price) {
        String sql2 = "select balance From account where username = ?";
        int balance = jdbcTemplate.queryForObject(sql2, Integer.class, userName);
        if (balance < price) {
            throw new UserAccountException("no money");
        }
        String sql = "update account set balance = balance - ? where username = ?";
        jdbcTemplate.update(sql, price, userName);
    }
}
```

service类：

``` java
public interface BookShopService {
    public void purchase(String username, String id);
}
```

service类实现：

``` java
@Service("bookShopService")
public class BookShopServiceImpl implements BookShopService {
    @Autowired
    private BookShopDao bookShopDao;
    @Transactional
    @Override
    public void purchase(String username, String id) {
        int price = bookShopDao.findBookPriceByID(id);
        bookShopDao.updateBookStock(id);
        bookShopDao.updateUserAccount(username, price);
    }
}
```

自定义库存异常：

``` java
public class BookStockException extends RuntimeException {
    public BookStockException(String message) {
        super(message);
    }
}
```


自定义钱不够异常：


``` java
public class UserAccountException extends RuntimeException {
    public UserAccountException(String message) {
        super(message);
    }
}
```

默认出现异常时回滚所有操作并抛出异常。