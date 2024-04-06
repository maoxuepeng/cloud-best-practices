---
date: 2020-2-15
title: 离线文字识别OCR工具
tags: [文字识别, OCR, tesseract]
---

文字识别的能力是AI应用的基础，几乎所有的云厂商都提供文字识别的API，使用者按调用次数付费。这种模式的成本较高，且速度慢、因为每次都要发起Internet请求。对于一些简单的文字识别需求，可以考虑使用离线工具替代。[tesseract](https://github.com/tesseract-ocr/tesseract) 就是一个优秀的离线文字识别工具。

## 介绍

[tesseract](https://github.com/tesseract-ocr/tesseract) 历史悠久，最初是由惠普公司研发的，1984-·995年间在惠普公司内部使用，1998年导入到Windows中使用，2005年开放源代码，2006年之后由谷歌维护开发。

[tesseract](https://github.com/tesseract-ocr/tesseract) 使用谷歌开源的 [leptonica](https://github.com/DanBloomberg/leptonica) 处理图像。

## 使用

[tesseract](https://github.com/tesseract-ocr/tesseract) 支持100+种语言，不同的语言通过外置语言包的形式，各种语言包可以从[这里下载](https://github.com/tesseract-ocr/tesseract/tree/master/tessdata) 。

[tesseract](https://github.com/tesseract-ocr/tesseract) 提供多种API被上层应用集成：命令行、SDK，图形界面形式由[三方开发者贡献](https://tesseract-ocr.github.io/tessdoc/User-Projects-%E2%80%93-3rdParty.html)。

命令行使用方式参考[Basic command line usage](https://tesseract-ocr.github.io/tessdoc/Command-Line-Usage.html)，典型的命令行格式：```tesseract <input-image> <output-file-base-name> -l [languages] --tessdata-dir [language-data-path]```

- input-image 需要识别的图像文件路径
- output-file-base-name 输出文件名不带后缀，生成文件名为 ```output-file-base-name.txt```
- [languages] 识别文字的语言，支持多种语言混合，如中英文混合则可输入: ```chi_sim+eng```
- [language-data-path] 语言包路径

## 效果

以唐诗“江雪”为例子 ```tesseract 江雪.png 江雪 -l chi_sim+eng --tessdata-dir tdata```

![江雪](/images/江雪.jpg)

识别的文字输出为：

```
 

江 雪
唐 代 : 柳 宗 元

干 山 鸟 飞 绝 , 万 径 人 踪 灭 。
孤 舟 群 笠 翁 , 独 钓 寒 江 雪 。

```

## Reference

[tesseract](https://github.com/tesseract-ocr/tesseract)

[leptonica](https://github.com/DanBloomberg/leptonica)
