---
title: 一些好用的工具
date: 2022-06-14 20:33:35
categories: 
	- tools
tags:
	- URI
	- Spring
---
## UriComponentsBuilder

`org.springframework.web.util.UriComponentsBuilder`

> spring带的解析uri工具

```java
    String url = "http://domain:port/path1/subpath1?param0=0&param1=1";

    UriComponentsBuilder.fromHttpUrl(url).build();
    UriComponentsBuilder.fromUri(URI.create(uri)).build();
    UriComponentsBuilder.fromUriString(uri).build();
    ....

    uriComponents.getHost(); // e.g. domain
    uriComponents.getPath(); // e.g. /path1/subpath1
    uriComponents.getPathSegments(); // e.g. [path1, subpath1]
    uriComponents.getScheme(); // e.g. http
    uriComponents.getQueryParams(); // [param0:0, param1:1]
```
