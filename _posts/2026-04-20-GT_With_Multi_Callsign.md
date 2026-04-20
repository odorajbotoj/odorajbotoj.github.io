---
layout: post
title: 单机多呼号配置 JTDX + GridTracker2 + TQSL
category: AmateurRadio
---

*JTDX + GridTracker2 + TQSL* 是一套很好的 *FT8* 通联工作流, 但有时我们需要在一台计算机上配置多个呼号并需要能快速切换. 我们可以利用 *GridTracker2* 的 **导入/导出配置文件** 来进行快速配置.

> 本文默认您会 *JTDX + GridTracker2 + TQSL* 的基本操作, 并且您已经配置好了自己呼号的工作流
>
> 作者使用的操作系统是 `Debian GNU/Linux 13 (trixie) x86_64`

本文将分文三个部分

+ TQSL 导入额外的呼号证书
+ GridTracker2 为不同的呼号建立配置
+ JTDX 切换呼号

## 1. TQSL 导入额外的呼号证书

打开 TQSL 软件, 点击 *呼号证书 -> 导入呼号证书* 进行导入. ( 一般使用 `.p12` 文件 )

![import](https://raw.githubusercontent.com/odorajbotoj/odorajbotoj.github.io/refs/heads/main/assets/img/2026-04-20-GT_With_Multi_Callsign_import.jpg)

然后回到 *台站位置*, 建立新的位置, 并使用新导入的呼号. **注意: 名称不要和之前的重复, 即使绑定的呼号不一样**

![location](https://raw.githubusercontent.com/odorajbotoj/odorajbotoj.github.io/refs/heads/main/assets/img/2026-04-20-GT_With_Multi_Callsign_location.jpg)

## 2. GridTracker2 为不同的呼号建立配置

打开 GridTracker2 软件, 打开设置, 首先在 *常规* 页点击 *导出设置* , 将自己的设置备份.

*查找* 设置和 *地图* 设置可以不变更, 这不会有很大影响. 我们需要重点关注的是 *日志记录* 设置.

根据需求修改 *日志记录* 设置. 例如此处我只启用了备份, PSK-Report 和 LoTW. LoTW 配置中 *台站位置* 选择刚刚新建的位置. **若有重名的台站位置, 则可能无法正常工作**.

![gt2cfg](https://raw.githubusercontent.com/odorajbotoj/odorajbotoj.github.io/refs/heads/main/assets/img/2026-04-20-GT_With_Multi_Callsign_gt2cfg.jpg)

配置妥当后, 再次在 *常规* 页点击 *导出设置*, 保存一个新的配置文件. 现在你可以利用 *导入设置* 来快速切换配置文件, 实现多呼号的值机. ( 导入设置时, GridTracker2 软件会重启 )

## 3. JTDX 切换呼号

这一步最简单, 在 JTDX 中使用快捷键 `F2` 打开设置, 更改一下呼号即可.

现在就可以快速在两个不同呼号的 FT8 工作流之间切换了. 切换呼号时, 需要注意以下几点:

1. JTDX 呼号与 GridTracker2 配置文件匹配
2. 台站位置 ( 网格 ) 匹配

---

BG4QBF

> Project A.S.Y.A. // Across Signals, Yearning Attunement.
