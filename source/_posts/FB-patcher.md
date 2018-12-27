---
title: 【黑苹果显卡驱动】通过Device/properties 给Framebuffer打补丁一点经验
date: 2018-12-03 17:21:47
tags: [黑苹果,显卡驱动]
categories: 黑苹果
---
### 本文参考[Coffee Lake帧缓冲区补丁及UHD630 Coffee Lake ig-platform-id数据整理](https://blog.daliansky.net/Coffee-Lake-frame-buffer-patch-and-UHD630-Coffee-Lake-ig-platform-id-data-finishing.html)，算是对文章的一种补充吧！注意，本篇文章不适合小白阅读！

<!-- more -->

### 一，打这个补丁有什么用处？
打这个补丁就能成功驱动你的`核显`，让它正常工作。如果已经成功驱动了核显的就没必要看了。
### 二，准备工作
* 添加启动参数 -cdfon，删除启动参数 -disablegfxfirmware
*   删除`FakePCIID` `IntelGraphicsFixup`,`NvidiaGraphicsFixup`,`Shiki`和`CoreDisplayFixup`

*   关闭`Clover`里面关于`Graphics`注入的参数，这些参数包括：
    *   config.plist/Graphics/Inject/ATI=NO
    *   config.plist/Graphics/Inject/Intel=NO
    *   config.plist/Graphics/Inject/NVidia=NO
    *   config.plist/Graphics/ig-platform-id=
    *   config.plist/Devices/FakeID/IntelGFX=

*   关闭`Clover`里面关于`DSDT`的修复：
    *   AddHDMI
    *   FixDisplay
    *   FixIntelGfx
    *   AddIMEI

*   禁用`UseIntelHDMI`

*   移除`boot argument`参数：`-disablegfxfirmware`

*   移除`IGPU`和`HDMI`部分的全部内容，包括：
    *   config.plist/Devices/Arbitrary
    *   config.plist/Devices/Properties
    *   config.plist/Devices/AddProperties

*   从CLOVER/ACPI/patched删除任何与`IGPU`和`HDMI`相关的`SSDT`和`DSDT`

* 下载`WhateverGreen`和`Lilu`最新版本
[Lilu下载地址](https://github.com/acidanthera/Lilu/releases)
[WhateverGreen下载地址](https://github.com/acidanthera/WhateverGreen/releases)

### 三，确定获取iGPU显卡设备的路径
下载并使用[gfxutil](https://github.com/acidanthera/gfxutil/releases)工具，如下所示：
```
$ gfxutil -f IGPU
DevicePath = PciRoot(0x0)/Pci(0x2,0x0)
```
这样我们确定了显卡路径之后，把`=`号之后的路径复制下来，填入如下图的所示的位置：
![数据填入展示](https://upload-images.jianshu.io/upload_images/8654767-057f4a46b7255d48.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 四，一些参数介绍（除了第6个值必须设置，其他可选）
1. framebuffer-patch-enable（是否启用framebuffer补丁，当然启用啊，不启用的话这篇文章还有什么用处）：
- DATA数据：01000000 -> 1（启用）    00000000 -> 0（不启用）
- NUMBER数据：0（不启用）   1（启用）

2. framebuffer-stolenmem（给BIOS中DVMT添加一点内存大小，会影响高分屏，这个值必须大于32M，也不应该过高）：
- 一般1080P屏幕的话，设置为48M就够用了：00003001
- 当你的笔记本电脑屏幕是2k，你可以设置为64M：00000004
- 4K屏的话，要设置为128M：00000008
如果你的BIOS中可以设置DVMT的话并且你设置成为128M之后，可以不需要设置这个属性，或者这个属性设置小一点：00003001
保险起见，高分屏直接设置成128M比较稳，并且保证在BIOS能设置DVMT的情况下设置在64M或以下
（PS：这一部分可能有误，但是最后一句保险起见，高分屏直接设置成128M比较稳是试验过的）

3. framebuffer-unifiedmem（核显显存大小，调大一点可能能解决花屏）：
- 2048M：00000080
- 3072M：000000C0

4. framebuffer-cursormem（翻译成中文就是光标内存，会影响高分屏，比如高分屏花屏可能就是这个值不够大）：
- 一般屏幕设置成9M大小就好：00009000
- 高分屏的话最好直接设置成48M：00000003 

5. framebuffer-fbmem（framebuffer内存大小，会影响高分屏）：
- 一般屏幕设置成9M大小就好：00009000
- 高分屏的话最好直接设置成48M：00000003

6. AAPL,ig-platform-id（设备平台id，直接影响显卡是否能成功驱动）：
举例一些常用`笔记本`的核显id（PS：如果没有列举您的，还望自己爬帖查找，一般别人制作的原版镜像也会提供多个核显配置文件供你们使用，在里面Graphics/ig-platform-id也可以看到。或者您还可以参考我文章开头提供的文章链接查找）：
- HD4600，HD4200，HD4000，HD5000，HD5100，HD5200：0a260006（如果不行设置后者），0a2e0008
- HD5300，HD5500，HD6000：16260006
- HD630：3e9b0000

7. device-id（设备id，可能是能让黑苹果正确显示设备信息，上面设备平台id一样的统一设置一个值）：
- 0a260006，0a2e0008：12040000
- 16260006：16160000
- 3e9b0000：9b3e0000
PS：本文没有收录的可以使用Intel FB Patcher这个软件查询，或者直接使用这个软件打补丁。具体用法：按照[这篇文章](https://blog.daliansky.net/Intel-FB-Patcher-tutorial-and-insertion-pose.html)成功输出config.plist之后，把你正在使用的config文件中Devices/Properties中全部的参数和值删除，然后把输出的配置文件对应的参数与值复制过去。[视频演示](https://www.bilibili.com/video/av35104213?from=search&seid=4599094922106870017)

8. framebuffer-conN-enable（N为数字，显卡第N个输出接口是否启用，1为启用，0为不启用）：
- DATA数据：01000000 -> 1（启用）    00000000 -> 0（不启用）
- NUMBER数据：0（不启用）   1（启用）

9. framebuffer-conN-type（N为数字，显卡第N个输出接口的类型）：
* 00080000 ：HDMI输出
* 0004000：DP输出（好像是的吧，记不清）

10. framebuffer-conN-index（个人理解，显卡第N个输出接口的优先级，或者说是设置第N个输出口的位置）：
这个按个人需要设置，如果需要屏蔽这个输出口，可以设置成FFFFFFFF，也就是最大的数字，让它足够靠后，这样就达到了屏蔽效果！

* 最后，请注意，所有DATA数据类型需要将数据两两一组倒过来填入，例如：16260006转换之后就是这样06002616，如下图：
![数据的填入](https://upload-images.jianshu.io/upload_images/8654767-53c6a2347ef503ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
所以你也可以发现，用framebuffer-fbmem参数举例，当需要设置为48M之后它应填入的值是：`00000003`，这个也是转换后的值，所以原来的值应当是`03000000`，这是一个16进制的数字，转换成10进制是`50331648`。我们知道1M=1024KB，1KB = 1024B，所以，我们把转换成十进制之后的数字`50331648`除以1024然后再除以1024，得出的结果就是48了，所以这串数字代表的就是48M。[点击这里前往进制转换网页](https://tool.lu/hexconvert/)
当然为了方便，你也可以直接像下图中切换成NUMBER数据类型，这样你就不用转换成16进制，不用倒过来输入（ig-platform必须为DATA）：
![转换数据类型](https://upload-images.jianshu.io/upload_images/8654767-803da1810195ac8d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 后文
本篇文章并不全面，还有一些参数没研究懂，毕竟黑苹果文化博大精深，所以当作者学习到新知识之后会不定期更新。喜欢的朋友可以点一波爱心，再顺手关注一下作者！