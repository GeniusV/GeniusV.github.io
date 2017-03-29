---
layout:     post
title:      "What is RESTful"
subtitle:   ""
date:       2017-03-29
author:     "GeniusV"
header-img: "img/date-a-live-257.jpg"
catalog: true
tags:
    - RESTful
---
REST:Representation State Transfer.

For exanple, if you want to browse someone's blog, you will tell the sever "I want the list of articles".Then, the server will return the resource back. After that, you say "I want to watch the first article". The server will return the first page of the first article. But when you request the second page, you can say "I want the next page". This is the point, RESTful server is stateless, which means it does't know which page and which article you request. So, when the server returns the first page of the article, it will return a text contains the URL of the current aricle. Then when you request the next page, your client have to tell the server like this: "I want the second page of the first article." In short, that means you client will storage the current state.

Define URLs like this:

```
GET /products : will return the list of all products
POST /products : will add a product to the collection
GET /products/4 : will retrieve product #4
PATCH/PUT /products/4 : will update product #4
```

NOT like this:

```
/getProducts
/listOrders
/retrieveClientByOrder?orderId=1
```
