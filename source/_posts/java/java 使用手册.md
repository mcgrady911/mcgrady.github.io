---
title: java 使用手册
date: 2018-01-23 21:30:50
updated: 2018-01-23 21:30:50
tags: [tips]
categories: java
---

# java 使用手册



### HashMap

1. 推荐的HashMap遍历方式

   ```java
   Map map = new HashMap();
   Iterator iter = map.entrySet().iterator();
   while (iter.hasNext()) {
     Map.Entry entry = (Map.Entry) iter.next();
     Object key = entry.getKey();
     Object val = entry.getValue();
   }
   ```

   ​