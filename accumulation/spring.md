---
title: spring积累
description: 
published: true
date: 2022-07-20T03:08:16.638Z
tags: spring
editor: markdown
dateCreated: 2022-04-30T14:25:42.394Z
---

## 如何自动配置依赖的项目的bean

只需要在被依赖项目中resource下添加META-INFO文件夹，在文件夹中添加文件spring.factories

文件内容：

```plaintext
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.qq.common.system.service.impl.SysAccountServiceImpl,\
  com.qq.common.system.service.impl.SysMenuServiceImpl
```

此时启动项目时会自动扫描并装配已经配置的bean