---
layout:     post
title:      "Kill Process"
subtitle:   ""
date:       2017-01-19
author:     "GeniusV"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Windows
    - cmd
---
Force to kill the process 2472

```
tasklist|findstr 2472
taskkill /pid 2472 -t -f
```