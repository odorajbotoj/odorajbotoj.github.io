---
layout: post
title: Linux上使用SDR接收卫星SSTV并解码
category: AmateurRadio
---

使用设备是 RTL-SDR V4 ，软件 SDR++ 、 qsstv 。天线用了根自制的 UHF 7-Elements YAGI ，效果还不错。

打开 SDR++ ，调整带宽至最小（应对卫星的多普勒绰绰有余），根据需要选择增益。

![set1](https://raw.githubusercontent.com/odorajbotoj/odorajbotoj.github.io/refs/heads/main/assets/img/2026-02-09-SDR_Recv_SSTV_set1.jpg)

将频谱的中心频率调到卫星过境时的最低频率（不然后期重放基带遇到负频率很难受），然后在卫星过境时录制基带。

然后，根据需求处理基带文件。这里我直接用 SDR++ "File" 模式重放了基带，并且把有信号的三十几秒单独录制了一遍，相当于截取中间段，节省存储空间。

然后就是（继续）重放基带，但这次我们要选择音频录制，来将 SSTV 的信号提取出来。

![set2](https://raw.githubusercontent.com/odorajbotoj/odorajbotoj.github.io/refs/heads/main/assets/img/2026-02-09-SDR_Recv_SSTV_set2.jpg)

根据瀑布图的显示对准频率。

![waterfall](https://raw.githubusercontent.com/odorajbotoj/odorajbotoj.github.io/refs/heads/main/assets/img/2026-02-09-SDR_Recv_SSTV_waterfall.jpg)

录制出来的音频也是 wav 格式的。现在我们可以打开 qsstv 来解码了。我们把 qsstv 的输入调成文件。

![set3](https://raw.githubusercontent.com/odorajbotoj/odorajbotoj.github.io/refs/heads/main/assets/img/2026-02-09-SDR_Recv_SSTV_set3.jpg)

点击 "Start Receiver" 就会有一个选择文件的窗口，我们选择之前提取的音频。 qsstv 解码音频不需要播放，而是直接数字信号处理大法，所以速度很快。

![qsstv](https://raw.githubusercontent.com/odorajbotoj/odorajbotoj.github.io/refs/heads/main/assets/img/2026-02-09-SDR_Recv_SSTV_qsstv.jpg)

这样就完成解码了，信号被接受之后全过程几乎无损，比手台录音强。
