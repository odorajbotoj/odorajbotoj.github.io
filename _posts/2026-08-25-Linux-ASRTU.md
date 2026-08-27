---
layout: post
title: Linux 下的 ASRTU-1 遥测接收工作流
category: AmateurRadio
---

> 本篇中, 笔者使用的系统为 `Debian 13 x64`
>
> 笔者默认读者熟悉 linux 相关操作.
> 
> 当你不确定自己是否需要 linux 时, 保持现状就好.
>
> 本文仅提供一种方法, 供交流学习使用

ASRTU-1 (下文简称为阿斯图) 是哈工大紫丁香卫星团队的一颗星, 其 UHF 遥测下行为 BPSK 9600. 其遥测制式及地面站工作流也被复用在最近发射的 BY-04 等卫星上. 以往, 阿斯图的接收与解调依赖 *GNU Radio* 等软件, 且主要工作在 *Windows* 平台上. Linux 平台虽有 LiveCD 可以使用, 但也面临效率底下/主系统老旧/操作复杂等问题. 笔者的主要工作与生活全部都在 Linux 上, 为了也能愉快收星, 我做了一些适配与探索, 在此形成一份文档, 供有需要的同志参考.

## 1. sdrpp 的魔改与使用

[AlexandreRouma/SDRPlusPlus](https://github.com/AlexandreRouma/SDRPlusPlus) 是一个跨平台的 sdr 软件, 交互简单, 界面直观, 但有些功能上不尽如人意. 我们需要使用 sdr 来接收阿斯图的下行, 软件方面在 linux 上很难有更好的选择了. 对原版 sdrpp 进行小修改之后使用, 可以达到不错的效果, 那就先用着.

### 魔改

sdrpp 接收阿斯图的一大问题是, 他的 USB 模式带宽不太够 (12000), 而他的 RAW 格式又不那么好用 (不太兼容后续工作流). sdrpp 的带宽设置非常保守, 其作者在 Issues [#392](https://github.com/AlexandreRouma/SDRPlusPlus/issues/392) [#958](https://github.com/AlexandreRouma/SDRPlusPlus/issues/958) 及 PRs [#1203](https://github.com/AlexandreRouma/SDRPlusPlus/pull/1203) 等中多次反对增加带宽, 并直言

> I didn't see the need for over 12KHz in USB, if you need to export the IQ for another decoder use the RAW mode instead.

> Wasting CPU usage by running the demodulators at a higher samplerate for no reason or dramatically increasing complexity and and decreasing responsiveness by dynamically changing the samplerate at which the demodulators run?
>
> There is no reason to have the modes go wider than actually needed because they are configured for normal usage. If you're trying to receive a complete out of spec signal it likely makes much more sense to have a dedicated demodulator.

然后收获了不少 :thumbsdown: .

所以我们只能修改其源代码并重新编译, 以支持更高的带宽, 来接入阿斯图解调流.

| 路径 | 原始值 | 修改值 |
| --- | --- | --- |
| `decoder_modules/radio/src/demodulators/radio_module.h` | `nb.init(NULL, 500.0 / 24000.0, 10.0);` | `nb.init(NULL, 500.0 / 48000.0, 10.0);` |
| `decoder_modules/radio/src/demodulators/dsb.h`, `decoder_modules/radio/src/demodulators/lsb.h`, `decoder_modules/radio/src/demodulators/usb.h` |  `double getIFSampleRate() { return 24000.0; }` | `double getIFSampleRate() { return 48000.0; }` |

现在就可以正常使用 24000 带宽的 USB 模式了.

### 使用

sdrpp 左侧菜单栏的组件, 可以在 **Module Manager** 中管理. 我个人习惯的配置是使用 **两个** Recorder 来分别录制 Baseband 和 Audio, 他们可以同时工作并且没有什么问题. 我还开启了 **Rigctl Server**, 用于联动 **Gpredict** 进行自动多普勒控制.

![sdrpp](https://raw.githubusercontent.com/odorajbotoj/odorajbotoj.github.io/refs/heads/main/assets/img/2026-08-25-Linux-ASRTU_sdrpp.png)

## 2. Gpredict 自动追踪

**Gpredict** 是 Linux 下一个比较好用的卫星预测软件. 笔者认为, 至少在 UI 上, 比 **Orbitron** 先进太多. Gpredict 的配置网上有大把资料, 这里不详细赘述, 只简单讲一下流程.

1. 新建一个自己的 QTH, 注意设置经纬度
2. 更新星历. 欢迎使用 [BI4PYM](https://github.com/BI4PYM) 老师的 [AutoTLE](https://github.com/BI4PYM/AutoTLE)
   - 如果星历总是更新失败, 尝试去 `$HOME/.config/Gpredict/satdata/` 中删除缓存
3. 添加想要跟踪的卫星. 选择对应的卫星, 可以查看下一次过境详情及后续过境简报.
4. *Gpredict Radio Control* 可以连接 `rigctld` 进行多普勒控制, 这里可以和之前配置的 sdrpp 联动上.

![gpredict](https://raw.githubusercontent.com/odorajbotoj/odorajbotoj.github.io/refs/heads/main/assets/img/2026-08-25-Linux-ASRTU_gpredict.png)

## 3. soundmodem BPSK 解调

能进行阿斯图解调的工具主要有

- grc 流图 (官方)
- [soundmodem](https://github.com/CLA-179/Lilacsat-soundmodem-CLi) (官方)
- (WIP) [OpenHoshimi](https://github.com/HyacinthSat/OpenHoshimi) ( [HyacinthSat](https://github.com/HyacinthSat) )

在 linux 上使用最方便且运行最稳定的当属 soundmodem 了. 这里使用 HyacinthSat 团队 fork 优化过的 [soundmodem](https://github.com/HyacinthSat/Lilacsat-soundmodem-CLi). 编译可以直接按照 [README.md](https://github.com/HyacinthSat/Lilacsat-soundmodem-CLi#for-linux) 中的步骤来, 在 linux 下非常便捷.

soundmodem 默认使用音频输入, 开放一个 ZMQ 端口 (5555) 送出遥测数据给上传器, 开放一个额外的 TCP 端口 (9985) 方便连接其他的数据解析器.

笔者的系统中音频使用 *pipewire*, 所以我使用 *Helvum* 软件可以直接控制软件间的音频通路, 将 sdrpp 输出的音频接入 soundmodem.

![sndmdm](https://raw.githubusercontent.com/odorajbotoj/odorajbotoj.github.io/refs/heads/main/assets/img/2026-08-25-Linux-ASRTU_sndmdm.png)

## 4. asrtu-sndmdm-backend 解析遥测数据

> 如果你的目标不是阿斯图, 不要使用这个软件

这是笔者的一个项目, 可以自动加载遥测格式, 并解析收到的数据, 形成表格.

[HyacinthSat/asrtu-sndmdm-backend](https://github.com/HyacinthSat/asrtu-sndmdm-backend)

这个工具可以搭配上文中的 soundmodem 使用, 也可以搭配流图使用. 他会连接 TCP 9985 端口上的服务.

这个工具针对阿斯图 223 bytes 遥测数据有特殊优化, 会进行不完整遥测包的拼包输出.

## 5. proxy_mmt 进行遥测上传

> 阿斯图, 我们的卫星
>
> 易用
>
> 转发器 (CQ SAT...)
>
> 轻松上行
>
> 我们的通联方式 (TNX 59 73)
>
> 但一切都有代价 (姿态失稳, 对日失败)
>
> 不, 我的母线电压, 不~~
>
> -- LOWPOWER --
>
> 哈哈, 觉得眼熟? 这样的事情, 正在太空中发生, 此时此刻!
>
> 没准就是下一圈. 
>
> 也就是说, 除非你做出这一生中最重要的决定:
>
> 向所有人证明你拥有高超技术
>
> 来进行卫星观测.
>
> 加入:
>
> 遥测贡献者

> Become a hero. Become a legend. BECOME A CONTRIBUTOR

官方上传器为 **proxy_mmt_gui**, 使用 c 编写, 但在 linux 下面存在一点点兼容性 bug (可能原因是依赖库更新). LiveCD 中提取的可执行文件由于动态链接的问题, 不能使用. 这里我重新实现了一个 **proxy_mmt_go** 来解决 linux 下的遥测上传问题. 其网络行为与 **proxy_mmt_gui** 相同, 已验证过可以进行遥测上传. 配置文件使用 json, 允许使用 `-c` 参数指定其他的配置文件. 去掉了 TUI, 并有输出 log 到文件的功能. 具体用途与配置, 参见 [项目主页](https://github.com/HyacinthSat/proxy_mmt_go).

我不建议在 Windows 上使用这个工具. Windows 上的 **proxy_mmt_gui** 足够好用.

---

> BG4QBF. 73

> Project A.S.Y.A. // Across Signals, Yearning Attunement.
