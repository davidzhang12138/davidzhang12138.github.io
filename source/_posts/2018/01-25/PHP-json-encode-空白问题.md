---
title: PHP json_encode 空白问题
date: 2018-01-25 17:17:34
tags: [PHP,JSON]
categories: [PHP]
---

昨天在修改基于shopsn的项目的源码时遇到个GAY问题
最后返回数据时 整个页面空白页 但是又不报错
经过排查 定位到是json_encode的问题
<!-- more -->
于是使用 json_last_error 函数 返回最后错误码
![错误信息](/img/18-1-25/json_last_error.png)

发现是 JSON_ERROR_INF_OR_NAN 于是查看 果然有个NAN参数 悄悄混了进去 于是做了处理 完美解决