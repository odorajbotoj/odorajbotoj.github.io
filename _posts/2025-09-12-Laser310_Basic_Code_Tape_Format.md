---
layout: post
title: Laser310 Basic 程序磁带文件格式
category: laser310
---

关于我翻译的 Laser310 中文文档见[这个仓库](https://github.com/odorajbotoj/Laser310BasicMan-zh)

今天整理一下 Laser310 Basic 程序磁带存储格式。 demo 见[这个仓库](https://github.com/odorajbotoj/BASICEditor)

首先是参考资料： `Laser-The BASIC-Interpreter for Laser 110 210 310 and VZ200.` 和 `VZ300 MainUnitManual` 。这两本书在网上都能搜到扫描本。还参考了 `BitWise WAV2VZ` 项目生成的音频，但是并未翻阅其源码。

接下来是详细解析。

![laser310-wavfile.jpg](https://raw.githubusercontent.com/odorajbotoj/BASICEditor/refs/heads/main/laser310-wavfile.jpg)

Laser310 Basic 程序的磁带音频采用 **8 位单声道 & 22050 采样率** 格式存储二进制数据。其中 `0` 对应 **6帧0xFF + 6帧0x00 + 12帧0xFF + 12帧0x00** ， 而 `1` 对应 **6帧0xFF + 6帧0x00 + 6帧0xFF + 6帧0x00 + 6帧0xFF + 6帧0x00** 。（ 1 帧 8 位，即 `0xFF` 对应 `0b11111111` ， `0x00` 对应 `0b00000000` ）

文件结构含有以下部分：

## 1. 头部

文件头由 255 个 `0x80 (0b10000000)` 和 5 个 `0xFE (0b11111110)` 组成，用来让 Laser310 识别信号。

## 2. 文件类型标识符

1 个 `0xF0` 表示这是一个 Basic 程序。

## 3. 文件名

最大 15 字节（未验证）的文件名。可以是图像字符也可以是 ASCII 字符（ [表](https://github.com/odorajbotoj/Laser310BasicMan-zh/blob/main/Laser310-BASIC%E5%BF%AB%E9%80%9F%E4%B8%8A%E6%89%8B.md#%E5%AD%97%E7%AC%A6%E8%A1%A8) ）。

## 4. 分隔符

1 个 `0x00` 用于标志整个头部信息的结束。然后是 58 **帧** `0x00` 。

## 5. 程序始末地址

1. 两个字节的程序起始地址，小端序。一般地， Laser310 Basic 程序的存储起始地址是 `0x7AE9` ，在这里存储为 `0xE9 0x7A` 。
2. 然后是两个字节的程序末尾地址 +1 ，小端序。打个比方，示例程序存储到 `0x7B0A` 为止，在这里就需要存储为 `0x0B 0x7B` 。

## 6. Basic 程序的 HEX

1. 两个字节的下行程序起始地址，小端序。
2. 两个字节的 Basic 行号，小端序。
3. 一行程序的 HEX 。
4. 一个 `0x00` 标记一行的结束。

举例，现有一行：

```basic
10 PRINT "TEST"
```

首先确认行号 10 ，存储为 `0x0A 0x00` 。此时推断出的地址为：

```text
      0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F
7AE0 __ __ __ __ __ __ __ __ __ __ __ 0A 00 __ __ __
7AF0 __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __

```

然后是 `PRINT` ，对应的 Token 为 `0xB2` ，剩下的内容直接存储 ASCII 值。

```text
      0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F
7AE0 __ __ __ __ __ __ __ __ __ __ __ 0A 00 B2 20 22
7AF0 54 45 53 54 22 __ __ __ __ __ __ __ __ __ __ __

```

然后加上结束符 `0x00` ，并且得出下一行指令的起始地址，一起填进去。

```text
      0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F
7AE0 __ __ __ __ __ __ __ __ __ F6 7A 0A 00 B2 20 22
7AF0 54 45 53 54 22 00 __ __ __ __ __ __ __ __ __ __

```

一行一行生成就行了。指令的 Token 见 [demo](https://github.com/odorajbotoj/BASICEditor/blob/fa557a6ef5ca0a84ac9d899f69a115aeb0a277bc/BASICEditor.py#L22-L188)

## 7. 程序结束符

两个 `0x00` 。

## 8. 校验和

对 [5](#5-程序始末地址) , [6](#6-basic-程序的-hex) , [7](#7-程序结束符) 所有生成的字节值求和，取最低 2 字节，小端序存储。

## 9. 结束

20 个 `0x00` ，实测可省略。
