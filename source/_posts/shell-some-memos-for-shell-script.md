---
title: Shell脚本笔记
date: 2022-03-19 23:27:41
categories: 
	- 伸展运动
  - Linux
tags:
	- Linux
	- shell script
---

- 接受用户输入
  
  ```bash
  read var
  echo "input value is $var"
  ```

- 成功后执行
  
  ```bash
  rm somefile && echo "removed"
  ```

- 失败后执行
  
  ```bash
  rm filenotexists || echo "nothing removed"
  ```

- 重定向输出
  
  - `1`代表`stdout`, 标准输出
  
  - `2`代表`stderr`, 标准错误输出
  
  - `&`代表"按照相同方式输出"
  
  - `dev/null`是null文件，可以理解为一个“黑洞”设备，所有输入都会被丢弃
  
  ```bash
  echo "blabla" > dest.file
  
  # 输出所有输出到文件
  echo "blabla" 1 > dest.file
  
  # 只输出错误输出到文件
  rm filenotexists 2 > err.log
  
  # 输出所有输出到文件
  rm somefile 1 > dest.file 2>&1
  
  # 输出非错误输出到文件
  rm somefile 1 > dest.file 2 > dev/null
  
  # 创建两条管道
  rm somefile 1 > log 2 > log
  
  # 2的管道继承于1
  rm somefile 1 > log 2>&1
  ```

- 创建固定大小的空文件
  
  当`dev/zero`作为输入流的时候，会产生无限的0 (不是ASCII的'0')
  
  ```bash
  # input file is /dev/zero
  # output file is file
  # write 10 times
  # 1024 bytes every time
  # so it creates a 10k blank file
  dd if=/dev/zero of=file count=10 bs=1024
  ```

- if else
  
  ```bash
  a=10
  b=20
  if [ $a == $b ]
  then
     echo "a is equal to b"
  elif [ $a -gt $b ]
  then
     echo "a greater than b"
  elif [ $a -lt $b ]
  then
     echo "a less than b"
  else
     echo ""
  fi
  ```

- 追加到文件
  
  ```bash
  file1 >> file2
  echo "tails" >> file
  date >> file
  ```
