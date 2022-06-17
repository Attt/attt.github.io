---
title: Case Sensitive collation in MySQL
date: 2022-06-17 15:51:08
tags:
    - MySQL
    - charset
    - collation
categories:
    - MySQL
---

## utf8mb4字符集排序规则

在MySQL8.0.1之前，utf8mb4没有cs排序规则，大小写敏感的排序规则只能用`utf8mb4_bin`，然后通过类似于`SELECT ... ORDER BY column COLLATE utf8_croatian_ci.`的方法来得到更“human”的排序结果。（bin的排序方式按照类似ACSII顺序，比如大小写的字母不一定会在一起）
- [SELECT * FROM mytable WHERE name COLLATE utf8_bin ="azolia"](https://forums.mysql.com/read.php?103,156527,198794#msg-198794)
- [SELECT ... ORDER BY column COLLATE utf8_croatian_ci](https://forums.mysql.com/read.php?103,19380,200971#msg-200971)


MySQL8.0.1之后的版本支持utf8mb4字符集的大小写敏感排序规则（utf8_*_cs）
- [MySQL 8.0.1: Accent and case sensitive collations for utf8mb4](https://dev.mysql.com/blog-archive/mysql-8-0-1-accent-and-case-sensitive-collations-for-utf8mb4/)
- [New collations in MySQL 8.0.0](https://dev.mysql.com/blog-archive/new-collations-in-mysql-8-0-0/)
