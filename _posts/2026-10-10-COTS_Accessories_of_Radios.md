---
layout: post
title: COTS 在无线电台自制配件中的应用
category: AmateurRadio
---

> 本文未经作者允许, 严禁转载
>
> 首次发布于 BG4QBF 薛子豪 的个人博客

> 本文侧重点为短波设备

> 本文出现的一些商品图片仅为举例, 不作为任何购买建议. 文中图片来自于网络或电商平台, 并且不展示商家相关信息

随着业余无线电数字模式的发展, 无线电台不再是需要一个手咪就能玩转的设备了. 很多时候, 我们会配上一些额外的配件, 连接电脑, 使用程序来调制信号, 控制发射, 最终实现通联. 很多无线电台都有相应的控制接口和控制协议, 官方也有制造相应的配件. 但官方的控制线缆价格往往并不理想, 于是社区里出现了各种各样的自制配件. 我本人使用的 **Xiegu G90S** 和 **Yaesu FT-817ND** 也均使用自制的连接线配件.

在自制和教人自制的过程中, 我渐渐发现了一些问题: 有些友台动手能力稍弱, 可能连焊接的工具和经验都没有; 有的时候我自己也犯懒, 一些配件可能懒得打板或动手制作......这个时候, 我想到可以多使用一些 COTS (Commercial Off-The-Shelf, 商用现成品或技术) 来方便我的 DIY. 下面我列出我自己找到的一些合适的产品, 供大家参考.

## 1. 关于连接线

很多电台使用的都是 MiniDin-X (MD-X) 的接口, 如 **Xiegu G90S** 使用了 *MD-8*, **Yaesu FT-817ND** 同时使用了 *MD-6* 和 *MD-8*. 这类线缆在工业控制上是非常常见的, 所以直接搜索相应型号就可以了. 图中所示的是一根一端为端子的 *MD-8* 线缆. 当然, 你也可以买一根普通线缆剪成两半.

![MD-8](https://raw.githubusercontent.com/odorajbotoj/odorajbotoj.github.io/refs/heads/main/assets/img/2026-10-10-COTS_Accessories_of_Radios/MD-8.jpg)

## 2. 关于串口芯片

> **为什么是串口**
>
> 有些机器没有那么高级, 做不到一线通, 串口是业界常用且很好用的解决方案了.
>
> 一线通机器一般用一根 USB 数据线就可以实现控制与音频传输了. 这类机器的机主, 本文对您的帮助不会很大, 哈哈

现代大多数电台的控制应该都是使用 TTL 电平的串口了, 使用 CH340G 芯片就可以实现 USB 转 TTL. 在电商平台搜索, 会发现有些产品是自带线头 (杜邦线) 的, 我们可以直接使用这种线, 和上面说的连接线对接使用.

![USB2TTL](https://raw.githubusercontent.com/odorajbotoj/odorajbotoj.github.io/refs/heads/main/assets/img/2026-10-10-COTS_Accessories_of_Radios/USB2TTL.jpg)

![CH340](https://raw.githubusercontent.com/odorajbotoj/odorajbotoj.github.io/refs/heads/main/assets/img/2026-10-10-COTS_Accessories_of_Radios/CH340.jpg)

## 3. 关于免焊接线套

考虑到部分友台并没有什么焊接经验, 我这里列举一个常见的免焊接线套, 可以实现两根线的对接.

![connector](https://raw.githubusercontent.com/odorajbotoj/odorajbotoj.github.io/refs/heads/main/assets/img/2026-10-10-COTS_Accessories_of_Radios/connector.jpg)

## 4. 关于声卡

> **为什么要使用额外声卡**
>
> 1. 不是所有机器都有内置声卡
> 2. 电脑或手机自带的 3.5mm 接口并不适合用于连接电台, 错误的连接会导致损坏设备, 非对口的接头会造成无法使用 (典型: TRRS 四环 3.5mm 接口的两种标准定义不一样)

声卡的选择有很多, 搞无线电也用不上 HiFi 级别的设备, 一般会使用便宜管饱的 CM108 芯片声卡.

![CM108](https://raw.githubusercontent.com/odorajbotoj/odorajbotoj.github.io/refs/heads/main/assets/img/2026-10-10-COTS_Accessories_of_Radios/CM108.jpg)

## 5. 关于免焊音频头

我个人习惯是电台音频出母口, 声卡也是母口, 中间再用 AUX 音频线连接. 这里给出一个免焊的音频母口案例.

![3-5M](https://raw.githubusercontent.com/odorajbotoj/odorajbotoj.github.io/refs/heads/main/assets/img/2026-10-10-COTS_Accessories_of_Radios/3-5M.jpg)

## 6. 关于音频线

对于一些 SDR 架构的电台, 比如我的 **Xiegu G90S**, 音频输入和输出的增益都是可调的, 声卡直连机器一般不会有大问题. 但对于一些老机器, 比如我的 **Yaesu FT-817ND**, 中间要做一些音频衰减, 才能不使电台或声卡过载. 制作一个衰减器有点麻烦, 我想了想决定直接用这种中间带音量调节旋钮的 AUX 音频线.

![AUX](https://raw.githubusercontent.com/odorajbotoj/odorajbotoj.github.io/refs/heads/main/assets/img/2026-10-10-COTS_Accessories_of_Radios/AUX.jpg)

## 7. 关于电源诱骗器

对于一些 QRP 电台, 使用快充诱骗是能正常使用且非常方便的. 我在用这种可调电压的诱骗器, 对于我的 **G90S**, 我可以试试 15V 输出以减小电流, 而对于我的 **FT-817ND**, 我可以使用 12V 甚至 9V 来获得更高的效率.

> 是的, FT-817ND 的供电很 ~~恐怖~~ 厉害, 可以参考 [这篇 HelloCQ 的文章](https://www.hellocq.net/forum/read.php?tid=331174)

![QC](https://raw.githubusercontent.com/odorajbotoj/odorajbotoj.github.io/refs/heads/main/assets/img/2026-10-10-COTS_Accessories_of_Radios/QC.jpg)

## 8. 关于热缩管

热缩管可以包裹住一些元件和电路板, 防止意外搭接造成的短路, 也能使一些接头处更加美观. 根据自己的器件大小选择合适尺寸的热缩管, 并且注意好热缩比例.

![pipe](https://raw.githubusercontent.com/odorajbotoj/odorajbotoj.github.io/refs/heads/main/assets/img/2026-10-10-COTS_Accessories_of_Radios/pipe.jpg)

使用上述部件, 就可以组合出音频线, 控制线等配件. 至于 USB 隔离, 音频隔离, 有其他的现成器件可以采用. 我还没有使用到相关器件, 故不作列举. 文章的最后, 祝各位 DIY 顺利. 73.