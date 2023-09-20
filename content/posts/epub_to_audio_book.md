---
title: "epub电子书按章节转为audio book"
date: 2023-09-20T17:10:34+08:00
tags: ["epub", "audio book", "tts", "text to speech", "manual"]
draft: false
---

## epub to text

### clone *epub-to-text*

```shell
git clone https://github.com/Projet-TAMIS/epub-to-text.git
```

### install dependencies

先安装nodeJS。

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
- [rany2/edge-tts](https://github.com/rany2/edge-tts)