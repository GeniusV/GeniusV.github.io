---
layout:     post
title:      "NamedParameterJDBCTemplate"
subtitle:   ""
date:       2017-01-19
author:     "GeniusV"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - JDBC
    - SQL
    - Spring
    - Java
---

## NamedParameterJDBCTemplate

若有多个参数，则不用对应位置， 直接对应参数名。缺点时是麻烦  
DEMO：  
``` java
@Test
    public void testNamedParameterJDBCTemplate(){
        String sql = "insert into employees(last_name, e_mail, id) values (:lastName, :email, :deptID)";
        Map<String, Object> paraMap = new HashMap<String, Object>();
        paraMap.put("lastName", "FF");
        paraMap.put("email", "ff@fuck.com");
        paraMap.put("deptID", 9);
        namedParameterJdbcTemplate.update(sql, paraMap);
    }
```
