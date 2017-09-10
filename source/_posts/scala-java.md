---
title: scala-java兼容性问题
date: 2017-09-10 20:04:27
tags: 
- scala
- java
- 兼容性
categories:
- scala
- 编程语言
---

## java类型隐式转换成scala类型

```
import scala.collection.JavaConverters._
import scala.collection.JavaConversions._
```

## 类型强制转换

classOf[String]

例如  val log = LoggerFactory.getLogger(classOf[xxx])

## java变长参数传递

需要加:_* 例如

a.method(parameter:_*)