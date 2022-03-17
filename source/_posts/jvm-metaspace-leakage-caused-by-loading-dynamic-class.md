---
title: <JVM> Metaspace leakage caused by loading dynamic class
date: 2022-03-17 15:59:28
tags:
---

## cause

Define class with `ClassLoader#defineClass`, no matter the parameter `name` is specified or not, the HotSpot JVM always throws OOM in metaspace at last.

## behavior of `ClassLoader#defineClass`

![defineClass](0.png)

`ClassLoader#defineClass` is JVM native method, the major steps are:

1. parse class file and check if class file format is correct
   
   ![vm specification](1.png)

2. check `systemDictionary` with parameter `name` to determine if Klass/KlassHandler is already loaded

(pretend here some source codes XD)

But if step-2 exists, why the metaspace will be full?

## what actually happens when VM parsing class file

In step-1 mentioned above, HotSpot VM not only parses the class file but save the class data structure (class code, vtable, itable, etc.) to metaspace. so even if the class name is totally the same, there is duplicated class data generated in metaspace, and which is the metaspace OOM cause.

![native define class](2.png)

![find klass in dict](3.png)

![check if class already defined](4.png)



Why not just only parse the file without saving it to metaspace?

In my gusse:

- In java code, most of class definition is done by `ClassLoader#loadClass()` which checks the class name before vm method invoked.
  
  ![loadClass in java](5.png)

- There is not so many chances that class file format is bad (ClassFormatError), so it will be a large waste that parse class file and then dellocate the memory and then parse it again if system dictionary dosn't contain the same class handler.

## Class data GC in metaspace

Actually there is no specific GC behavior in metaspace, in VM the GC always works in heap, but the pointer pointed to class data in metaspace is saved in class instance which lives in heap. So if the class instance is time to be collected, the class data in metaspace will be freed to.
