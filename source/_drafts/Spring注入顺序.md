title: Spring注入顺序
author: Wtli
date: 2021-07-23 09:53:54
tags:
---
框架：SpringBoot
1. mybatis中引入mapper。
2. @service注解
3. @Service中引入mapper为null
4. @Service和@Component

<!--more-->

非Controller类，定义@Service类，在类中引入@Resource的mapper，在@Autowired中调用mapper，为null。