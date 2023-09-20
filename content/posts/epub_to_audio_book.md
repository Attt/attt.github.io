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

```javascript
'use strict';

var EPUBToText = require('./index');

var epubToText = new EPUBToText;
epubToText.extractTo('xxx/xxx.epub', 'xxx/output/', (err) => {
  // files are in folder, name according to the following convention:
  // sequence number + original epub file name + .txt
})
```

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

*epub2txt2*的参数`-a`(`--ascii`)可以忽略注音，这在很多语言里还是很实用的，比如日文的假名注音。这些注音如果不忽略的话在后续的tts处理时会影响效果（同样的词读两遍）。

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

## conclusion

the female Japanese voice and US voice of MS Edge is almost perfect. 🥸

---

***[参考]***

- [Projet-TAMIS/epub-to-text](https://github.com/Projet-TAMIS/epub-to-text.git)
- [kevinboone/epub2txt2](https://github.com/kevinboone/epub2txt2.git)
- [rany2/edge-tts](https://github.com/rany2/edge-tts)