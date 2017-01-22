---
layout:     post
title:      "Virtualenv 的使用"
subtitle:   ""
date:       2016-10-31
author:     "GeniusV"
header-img: "img/post-bg-2015.jpg"
tags:
    - python
---
## 环境的复制和迁移

虚拟环境如果直接复制会有问题，需要手动修改路径，而且并不保证配置正确。应该首先进入原环境,然后：
``` bash
pip freeze > requirement.txt
```
这样会在当前目录下创建一个名为`requirement.txt`的文本文件，把它放到目标机器上（目标机器必须装有Virtualenv）执行：
``` bash
pip install -r requirements.txt
```
pip会自动下载安装包。

