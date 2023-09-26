---
title: "用Spring的URL解析工具解析带fragment的URL时异常"
date: 2022-07-20T13:39:02+08:00
tags: ["Spring", "URI", "bug fix"]
draft: false
---

## 现象

用Spring的UriComponents解析带fragment的URL`www.xxxx.com/p#/f`的时候报`IllegalArgumentException`。

## 分析

* URI的组成：[RFC3986](https://datatracker.ietf.org/doc/html/rfc3986)

![RFC3986](/images/spring_url_util_parsing_issue/rfc3986_url_segments.png)

UriComponents的方法`fromHttpUrl(String)`执行时的匹配规则是：
- PATH部分不允许`?`或`#`出现
- `?`后作为一个整体

也就是`www.xxxx.com/p#/f`里`/p`被解析为了path部分，`#/f`是不合法的部分。

而`www.xxxx.com/p?#/f`这种是谋梦提的，因为`?#/f`部分被解析为一个整体。

![5.2.8之前的spring解析url的匹配规则](/images/spring_url_util_parsing_issue/uriComponents_parsing_regex.png)

当然这个问题实际上是fix(enhance)了的：

[`ISSUE#25300`](https://github.com/spring-projects/spring-framework/issues/25300):Support fragments in UriComponentsBuilder.fromHttpUrl()

`5.2.8.RELEASE`的spring已经将修复代码合并：
![commit](/images/spring_url_util_parsing_issue/spring-issue-commit.png)

## 解决

1. 升级spring到5.2.8.RELEASE及以上版本
2. 用`fromUriString(String)`替换`fromHttpUrl(String)`，再根据schema筛选