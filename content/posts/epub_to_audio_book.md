---
title: "epub电子书生成audio book"
date: 2023-09-20T17:10:34+08:00
tags: ["epub", "audio book", "tts", "text to speech", "manual"]
draft: false
---

## 使用[epub-to-text](https://github.com/Projet-TAMIS/epub-to-text.git)转epub文本

*epub-to-text*会自动根据章节拆分。

### clone *epub-to-text*

```shell
git clone https://github.com/Projet-TAMIS/epub-to-text.git
```

### install dependencies

需要先安装nodeJS

再拉依赖
```shell
npm install
```

### create `main.js`

{{< gist Attt 189815c084d51964c3de9980d40d909b >}}

### run with *node*

```shell
node main.js
```

## 使用[epub2txt2](https://github.com/kevinboone/epub2txt2.git)转epub文本

*epub-to-text*根据章节拆分有时候（大部分时候😅）不准。

### clone *epub2txt2*

```shell
git clone https://github.com/kevinboone/epub2txt2.git
```

### make & make install

需要先安装gcc和make

再编译

```shell
make & make install
```

### convert

~~*epub2txt2*的参数`-a`(`--ascii`)可以忽略注音，这在很多语言里还是很实用的，比如日文的假名注音。这些注音如果不忽略的话在后续的tts处理时会影响效果（同样的词读两遍）。~~（实测不灵🙅）

```shell
epub2txt -a xxx.epub > xxx.txt
```

### split txt file

拆分生成的txt

在linux环境下：

```shell
split -l 888 output.txt -d -a 2 output__
```

- `-l`: 单个文件行数
- `-d`: 以数字命名输出文件
- `-a`: 数字位数

## 使用[epub2splittxt](https://github.com/gtas5/epub2splittxt.git)转epub文本

这个仓库比较老，使用的是python2，而且不支持epub3。（epub3中不强制要求有toc.ncx作为目录）

功能上也不支持指定输出目录。

### clone *epub2splittxt*

```shell
git clone https://github.com/gtas5/epub2splittxt.git
```

### install pip2 for python2

```shell
curl https://bootstrap.pypa.io/pip/2.7/get-pip.py --output get-pip.py
```

### convert

```shell
python2 epub2splittxt.py a.epub
```


## text to speech

*edge-tts*基于Microsoft Edge TTS service

### install *edge-tts*

```shell
pip3 install edge-tts
```

### list avaiable voices (optional)

```shell
edge-tts --list-voices
```

### text to speech

```shell
edge-tts --voice ja-JP-NanamiNeural --text "$(cat xxx/ouput/chapter1.txt)" --write-media  xxx/ouput/chapter1.mp3 --write-subtitles xxx/ouput/chapter1.vtt
```

## 整合版

### clone [Attt/epub2audiobook](https://github.com/Attt/epub2audiobook)

```shell
git clone https://github.com/Attt/epub2audiobook
```

主要是优化了下[epub2splittxt](https://github.com/gtas5/epub2splittxt.git)：

1. 用python3重写
2. 支持epub3
3. 删除注音
4. 支持指定目录
5. 过滤掉可能的目录文本以及链接或者图片标记

### convert

一条龙转换：

```shell
./epub-tts.sh /path/to/epub.epub /path/to/output/
```

脚本：

{{< gist Attt e1d071f123411f0accb07def46be47d6 >}}

## conclusion

有一说一，MS的TTS是真的吊。 🥸

---

***[参考]***

- [Projet-TAMIS/epub-to-text](https://github.com/Projet-TAMIS/epub-to-text.git)
- [kevinboone/epub2txt2](https://github.com/kevinboone/epub2txt2.git)
- [rany2/edge-tts](https://github.com/rany2/edge-tts)
- [epub2splittxt](https://github.com/gtas5/epub2splittxt.git)
- [Attt/epub2audiobook](https://github.com/Attt/epub2audiobook)