---
title: <Shell> Some memos for shell script
date: 2022-03-19 23:27:41
tags:
---

- accept user input
  
  ```bash
  read var
  echo "input value is $var"
  ```

- do after pre-command succeed
  
  ```bash
  rm somefile && echo "removed"
  ```

- do after pre-command failed
  
  ```bash
  rm filenotexists || echo "nothing removed"
  ```

- redirect output
  
  - `1` means `stdout`, standard output
  
  - `2` means `stderr`, standard error output
  
  - `&` means "as the same way"
  
  - `dev/null` is a null file like a "blak hole", anything written to it is discard
  
  ```bash
  echo "blabla" > dest.file
  
  # complete
  echo "blabla" 1 > dest.file
  
  # error output to dest.file
  rm filenotexists 2 > err.log
  
  # all output to dest.file
  rm somefile 1 > dest.file 2>&1
  
  # only stdout
  rm somefile 1 > dest.file 2 > dev/null
  
  # create 2 pipes, overwrites each other
  rm somefile 1 > log 2 > log
  
  # 2 extends 1's pipe, works fine
  rm somefile 1 > log 2>&1
  ```

- create blank file with certain size
  
  `dev/zero` as an input stream provides endless 0s (not ASCII '0')
  
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

- append to file
  
  ```bash
  file1 >> file2
  echo "tails" >> file
  date >> file
  ```
