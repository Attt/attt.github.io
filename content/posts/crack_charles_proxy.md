---
title: "[JARHD破壊版]Charles Proxy"
date: 2024-03-15T15:13:02+08:00
tags: ["crack", "charles", "decompile", "java", "jar", "manual"]
draft: false
---

## 下载&安装[Charles Proxy](https://www.charlesproxy.com/download/)

![charles proxy download page](/images/crack_charles_proxy/scrshot01.jpg)

## 下载[JDI_GUI](https://java-decompiler.github.io/)

![JDI_GUI download page](/images/crack_charles_proxy/scrshot02.jpg)

## crack

### 認証Classの位置を把握

1. 取出`charles.jar`文件，用`JDI_GUI`打开。

位于：

- MacOS : `/Applications/Charles.app/Contents/Java`
- Win : 自分で探せ
- Linux : 自分で探せ

2. 找到`com/xk72/charles/Main.class`

看逻辑定位到认证类：

![JDI_GUI Main.class](/images/crack_charles_proxy/scrshot03.jpg)

比如这里是`p.b()`:

![JDI_GUI p.class](/images/crack_charles_proxy/scrshot04.jpg)
(no thx :D )

### `p.class`の中身を書き換え

1. 从`p.class`拷贝出所有的`public static`方法和字段到`$work_dir/p.java`, 保留一个空构造, 删掉所有import。

```java
package com.xk72.charles;

public class p {
  public static final String j = "no thanks.";
  
  public p() {}
  
  public static void j(p paramp) {
  }
  
  public static boolean j() {
    return true;
  }
  
  public static void a() {
  }
  
  public static String b() {
    return "attt";
  }
  
  public static String j(String paramString1, String paramString2) {
    return null;
  }
}
```

2. 编译`p.class`

```bash
javac --source 11  --target 11 -encoding UTF-8 p.java -d .
```

`--source`和`--target`用来指定编译的java版本


![java version](/images/crack_charles_proxy/scrshot05.jpg)


3. 打包好的`p.class`拷贝回`charles.jar`

```bash
jar -uvf ./charles.jar com/xk72/charles/p.class
```

### `charles.jar`を上書き

再用`JDI_GUI`检查一下，没问题就覆盖掉原来的`charles.jar`

![JDI_GUI modified p.class](/images/crack_charles_proxy/scrshot06.jpg)


***MacOS***

解决替换之后无法打开的问题（提示损坏之类的）

```bash
sudo xattr -rd com.apple.quarantine '/Applications/Charles.app'
```

## conclusion

![charles proxy](/images/crack_charles_proxy/scrshot07.jpg)

---

***[参考]***

- [100apps/charles-hacking](https://github.com/100apps/charles-hacking)


