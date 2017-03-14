---
layout:     post
title:      "How to create a web app in Idea with gradle"
subtitle:   ""
date:       2017-03-14
author:     "GeniusV"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - IDEA
---

1. choose File -> new -> project
2. choose Gradel
3. choose SDK
3. choose Java and Web on the right panel
4. choose next
5. enter GroupId and Aritfictid
6. change Version to 0.0.1-SNAPSHOT
7. choose next
8. choose Gradle JVM
9. choose next
10. confirm name and location
11. choose Finish
12. wait for gradle to complete
13. add servlet-dependency to build.gradle
14. choose Gradle and click refresh button
15. check if the dependency is installed
16. create a java dictonary in main dictionary, which shoudld become a root dictionary automatically.
17. create WEB-INF dictionary in webapp dictionary.
18. create web.xml in WEB-INF. If there is no template, choose Edit File Templates. Then, create a template in files, and copy Code -> Other -> Web -> Deployment descriptors -> Web.3_1.xml to the template and press OK
19. choose Edit Configuration (beside the run button), add a Local Tomcat Server. 
20. Configure the Tomcat Server if is necessary.
21. choose depolyment and add a artifact having a explored at the end.
22. the application context is recommened
23. Choose Server. On Update action and On frame deactication both choose Update classes and resources.
24. choose apply
25. Done !