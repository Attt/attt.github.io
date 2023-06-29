---
title: Google image search cause firefox crash
date: 2023-06-28 11:27:56
categories:
    - BUG草集
tags:
    - bug
    - AI
---


复现：

![用例](image.png)

[1839669 - Google Images search reproducibly causes tab crash - bugzilla](https://bugzilla.mozilla.org/show_bug.cgi?id=1839669)

原因：

```text
Google Images creates a huge stack frame with more than 19550 slots (more than 150 KB)
and then uses OSR to enter Baseline Interpreter code.

The stack probing we do there caused crashes because older kernels don't like it when
the distance between the address and RSP is more than about 64 KB.
```