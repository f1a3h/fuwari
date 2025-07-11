---
title: 记一次 OnePlus 9 Pro 刷入 Nameless 系统
published: 2025-03-15
description: '给 OnePlus 9 Pro 刷入原生 Android 15 系统 Nameless CLO'
image: ''
tags: [Android]
category: 'Yak'
draft: false 
lang: 'zh_CN'
---

## 前言

很久以前就知道 [Pixel Experience](https://get.pixelexperience.org/) 这个东西了，原生安卓 + Material Design, 看起来很香，加之当时手里只有一台从老爸那里继承来的退役 Huawei P10, 在未经同意的情况下自动升级到了 Harmony OS 系统，这导致我无法安装第三方来源的 Galgame 模拟器躺玩 Gal，一直很不爽这一点，于是想给它刷成 Pixel Experience 系统，结果华为不给解 BL 锁无法刷机，P10 遂吃灰至今。

前段时间闲来无事上咸鱼看了眼 Google Pixel 的价格，再看一眼硬件配置，实在是感人，但是一直都很馋安卓上丰富的开源软件，心血来潮之下就开始考虑刷机这个方案。在酷安上了解了一番，遂锁定方案为：

- 设备：OnePlus 9 Pro
	- [各厂商上市的手机型号与对应的宣传名汇总](https://khwang9883.github.io/MobileModels/)
	- 直接开放 Bootloader 解锁, 二手价格相对便宜，第三方 ROM 够丰富，硬件也够用
- 第三方 ROM: Android 15 版 Nameless CLO
	- Nameless OS 基于 氧 OS, 对一加手机支持较好
	- 艺不高但是人胆大，虽然没刷过机，但是也是小时候经常 ~~(一键)~~ Root 开作弊器打游戏的人，要刷就要刷最新的 ROM!
		- ~~差点刷成了砖~~
- 电脑系统：Windows 11

开刷！

啊等等，还是先来了解一下刷机相关知识点吧（怂）：

- [\[基础预科\] ADB、Android 终端、Android 用户权限](https://ak-ioi.com/714-preintro-adb/)
- [\[基础预科\] Android 分区、启动模式、Fastboot](https://ak-ioi.com/763-preintro-android-bootmode/)

## 准备工作

### 安装所需工具

电脑下载 [Platform Tools](https://dl.google.com/android/repository/platform-tools-latest-windows.zip) 以及 [Google USB Drivers](https://dl.google.com/android/repository/latest_usb_driver_windows.zip) 以便 USB 连接手机使用 adb 和 fastboot 等工具，后续使用 `adb` 和 `fastboot` 等命令时需要先 `cd` 进入相应程序所在文件夹，也就是下载的 Platform Tools 文件夹内。

不过我直接安装了 Android Studio 然后在 Android Studio 里面安装了这些必备工具：

```sh
scoop install android-studio
```

接着再找到 Android Studio 安装 SDK 所在目录并将其内的 platform tools (例如我的路径是：`C:\Users\<username>\AppData\Local\Android\Sdk\platform-tools`) 加入了环境变量，这样就不需要先 `cd` 进去才能用 `adb` 和 `fastboot` 了。

### 下载镜像文件

OnePlus 9/9 Pro 所需的 Android 15 版本 Nameless CLO 镜像于 2025 年 1 月 22 日制作完成，并于 1 月 23 日发布在 Nameless CLO 官方的 [Telegram 频道](https://t.me/NamelessUpdates), 暂未发布于官网上。

下载地址：[OnePlus 9](https://sourceforge.net/projects/nameless-clo/files/lemonade/Nameless-CLO-15.0-20250122-lemonade-Official.zip/download) | [OnePlus 9 Pro](https://sourceforge.net/projects/nameless-clo/files/lemonadep/Nameless-CLO-15.0-20250122-lemonadep-Official.zip/download)

另外，刷入该版本镜像前需仔细阅读 [Changelog](https://raw.githubusercontent.com/Nameless-CLO/android_OTA/refs/heads/fifteen/lemonadep/changelog_2025_01_22.txt).

## 解锁 Bootloader

首先，点击 7 次 "设置" -> "关于本机" -> "版本信息" -> "版本号" 进入开发者模式，接着进入 "设置" -> "其他设置" -> "开发者选项" 打开 USB 调试和 OEM 解锁。

然后手机使用 USB 线连接电脑，注意在手机上弹出的询问是否同意 USB 调试的消息框选择允许，接着打开 CMD 后使用以下命令检查一下 `adb` 是否能检测到设备：

```sh
adb devices
```

若一切正常，它应该会产生类似如下的输出：

```sh
List of devices attached
ab5ccfdf        device
```

然后使用命令 `adb reboot bootloader` 来重启手机进入 FastBoot。

下图为 OnePlus 9 Pro 的 FastBoot 界面，其中 "START" 为重启进入系统，未解锁 Bootloader 的情况下 "DEVICE STATE" 应为 "locked", 这里由于是我解锁之后补拍的因此显示为 "unlocked"。FastBoot 界面需要使用音量上下键进行选择，电源键进行确认。

![FastBoot](https://picgo-1259588753.cos.ap-beijing.myqcloud.com/202503151725562.jpg)

需要注意的是，由于刷机过程中可能会出现失误导致无法开机，继而无法通过 USB 调试进入 FastBoot 进行修复, 因此我们仍需学习如何在关机界面通过组合键直接进入 FastBoot, 建议在刷机前先尝试该方法并成功至少一次后再进行刷机。以下操作[^1]针对 OnePlus 设备，对于其他设备请自行 Google.

较老设备：

|     组合键（长按）      |     功能      |
| :--------------: | :---------: |
|    电源 + 音量上键     | Recovery 模式 |
|    电源 + 音量下键     | FastBoot 模式 |
| 电源 + 音量上键 + 音量下键 | Nothing(?)  |

较新设备：

|     组合键（长按）      |                                 功能                                 |
| :--------------: | :----------------------------------------------------------------: |
|    电源 + 音量上键     |                                正常启动                                |
|    电源 + 音量下键     | 1. Bootloader 未解锁，为 Recovery 模式<br>2. Bootloader 已解锁，为 Fastboot 模式 |
| 电源 + 音量上键 + 音量下键 |                            FastBoot 模式                             |

等待手机重启成功并被电脑检测到后，使用如下命令检测 `fastboot` 是否能检测到设备：

```sh
fastboot devices
```

若一切正常，它应该会产生类似如下的输出：

```sh
ab5ccfdf        fastboot
```

若这里没有任何输出，则进行如下步骤：

1. 在电脑上打开设备管理器；
2. 找到图标中带有黄色 '?' 标识的 Android 设备，例如我的出现在 "其他设备" 这一栏，选中后右键选择“更新驱动程序”；
3. 依次选择“浏览我的电脑以查找驱动程序” -> “让我从计算机的可用驱动程序列表中选取”；
	1. 若此处没有可用的驱动程序，说明系统缺失 ADB 驱动，对于 Windows 11 可以在电脑的设置中的 "检查 Windows 更新" -> "高级选项" -> "可选更新" 中查看是否存在 ADB 驱动，若有则安装即可，若没有则需要自行在网上下载安装。
4. 选择 Android Composite ADB Interface 进行安装。

完成后，在设备管理器中可以看到设备被识别为 Android Composite ADB Interface:

![Android Composite ADB Interface](https://picgo-1259588753.cos.ap-beijing.myqcloud.com/202503151755349.png)

然后再使用 `fastboot devices` 命令可以得到正常输出。

:::warning
注意刷机过程中请确保手机与电脑之间的 USB 连接完好，否则可能会出现无法开机的结果。
:::

接着，使用命令 `fastboot flashing unlock` 进行解锁，此时设备上会弹出警告信息，仔细阅读后若确认要解锁则使用音量键选中 "UNLOCK THE BOOTLOADER", 再使用电源键确认即可解锁。

:::warning
解锁会清除所有设备中所有设备数据，请注意备份。
:::

将设备重启后再次进入 FastBoot 界面，若 "DEVICE STATE" 显示为 "unlocked" 说明解锁成功。此后，开机会出现 Orange State 警告。

:::note
在某些设备上可能需要使用 `fastboot oem unlock` 命令进行解锁。
:::

:::note
使用命令 `fastboot flashing relock` 可以重新上锁，然而若设备镜像未恢复原厂状态，重新上锁会导致无法开机。
:::

## 刷入镜像

重启设备进入 FastBoot 界面，打开 CMD 并 `cd` 进入下载的镜像文件所在文件夹，使用下述命令刷入：

```sh
fastboot flash vendor_boot vendor_boot.img # 刷入供应商启动镜像
fastboot flash boot boot.img               # 更新内核和 ramdisk
fastboot flash dtbo dtbo.img               # 更新设备树
fastboot wipe-super super_empty.img        # 重置动态分区
```

镜像刷入完成后重启设备进入 Recovery, 可以选择使用 `adb reboot recovery` 或者关机界面按组合键进入 FastBoot 后使用音量键选择进入。

![Recovery mode 界面](https://picgo-1259588753.cos.ap-beijing.myqcloud.com/202503151845967.jpg)

依次选择 "Install Update" -> "ADB Sideload" 开启侧载，然后打开 CMD 使用下述命令推送 Nameless CLO 的 ROM 包：

```sh
adb sideload <filename>.zip
```

若出现 `adb cannot read '<filename>.zip'` 错误，则将 ROM 包移动到与 `adb` 程序同一路径（例如 `C:\Android`）下[^2]，再次使用上述命令进行侧载即可成功。

耐心等待侧载完成，当设备左下角显示进度为 47% 时电脑上会停住，同时设备中会提示是否现在 format data, 此时先不 format 而是等待 sideload 完成。

最后再次进入 Recovery, 依次选择 "Factory reset" -> "Format data/factory reset" -> "Format data" 格式化数据后，重启设备进入系统即可看到成功刷入 Nameless CLO 系统。

![Welcom to Nameless CLO!](https://picgo-1259588753.cos.ap-beijing.myqcloud.com/202503151906095.jpg)

:::warning
第一次进入系统等待时间会比较长。
:::

## 玩机

### 刷入 Magisk

在手机上安装好 Magisk APP, 然后连接电脑将之前刷入的 boot.img 文件传入手机中，再在手机中打开 Magisk, 选择 "Install" -> "Select and Patch a File", 并选择文件为电脑传入的 boot.img 文件，等待 Magisk 完成镜像修补后会得到修补后的镜像文件，将其传入电脑以便后续刷入。

![修补镜像](https://picgo-1259588753.cos.ap-beijing.myqcloud.com/202503151917662.jpg)

在新系统中若需要使用 USB 调试功能，同样需要先启用开发者模式并 enable USB debugging, Nameless CLO 系统启用开发者模式的方式为点击 7 次 "Settings" -> "About phone" -> "Build number".

接着重启系统进入 FastBoot 再使用命令 `fastboot flash boot <patched_boot>.img` 刷入修补后的镜像再重启进入系统即完成刷入，在实际刷入前可以先使用命令 `fastboot boot <patched_boot>.img` 对新的镜像进行试用。

:::note
`fastboot flash boot` 是实际刷入镜像文件，而 `fastboot boot` 则是临时以该镜像文件启动。
:::

此后打开 Magisk APP, 若 "Magisk" 下 "Install" 项显示为 Magisk 的版本号则说明刷入成功。

对于一些实用的 Magisk + Xposed 模块可以参考文章 [基于 HyperOS 2.0 和 Android 15 的个人 Magisk + Xposed 模块分享 - HikariLan's Blog](https://blog.hikarilan.life/thinking/1909/%E5%9F%BA%E4%BA%8E-hyperos-2-0-%E5%92%8C-android-15-%E7%9A%84%E4%B8%AA%E4%BA%BA-magisk-xposed-%E6%A8%A1%E5%9D%97%E5%88%86%E4%BA%AB/).

### 实用开源软件

- Android FOSS (Free and Open Source Software) 收集：[F-Droid](https://f-droid.org/)
- YouTube: [LibreTube](https://github.com/libre-tube/LibreTube)
- Bilibili: [PiliPalaX](https://github.com/orz12/PiliPalaX) 或 [bilimiao](https://github.com/10miaomiao/bilimiao2)
- 视频播放器：[Reex](https://gitee.com/lntls/reex)
- Pixiv: [PixEz Flutter](https://github.com/Notsfsssf/pixez-flutter)
- 阅读 3.0：[Legado](https://github.com/gedoor/legado)
- 知乎：[Hydrogen](https://github.com/Xmader/hydrogen)
- RSS 阅读器：[Read You](https://github.com/Ashinch/ReadYou)
- V2EX: [VVEX](https://github.com/hanai/vvex)
- 酷安：[c001apk](https://github.com/bggRGjQaUbCoE/c001apk)
- 贴吧：[贴吧 Lite](https://github.com/HuanCheng65/TiebaLite)
- 音频播放器：[Metro](https://github.com/MuntashirAkon/Metro)
- 视频/音频下载器：[Seal](https://github.com/JunkFood02/Seal)
- 游戏串流：[Sunshine](https://github.com/LizardByte/Sunshine)

### ACGN 相关

漫画：

- [pikapika](https://github.com/niuhuan/pikapika)
- [Mihon](https://github.com/mihonapp/mihon)
- [Kotatsu](https://github.com/KotatsuApp/Kotatsu)
- [copymanga](https://github.com/fumiama/copymanga)

番剧：

- [Aniyomi](https://github.com/aniyomiorg/aniyomi)
- [Miru](https://github.com/miru-project/miru-app)
- [Kazumi](https://github.com/Predidit/Kazumi)

轻小说：

- [LNReader](https://github.com/LNReader/lnreader)
- [LightNovelReader](https://github.com/dmzz-yyhyy/LightNovelReader)

Bangumi:

- [Bangumi for Android](https://github.com/xiaoyvyv/bangumi)

Galgame 模拟器：

- Tyranor
- [Kirikiroid2](https://github.com/zeas2/Kirikiroid2)
- [ONScripter Plus](https://play.google.com/store/apps/details?id=com.onscripter.plus&hl=en_NZ)
- [winlator](https://github.com/brunodev85/winlator)
	- 需手动安装下列工具才可愉快游玩大部分汉化版 Gal[^3] :
		- Chs Fonts(zh_CN).exe: 显示中文字体
		- main.1.com.winlator.obb: obb数据包
		- setup.exe: 微软运行库
	- 亦可选择下载大佬整合好的 mod 版开机即用，例如 [Winlator-Amod](https://github.com/afeimod/winlator-mod)

## 参考资料

参考资料？致谢！

- 著名搞机论坛 [XDA](https://xdaforums.com/)
- 官方 Wiki [Nameless CLO | OnePlus 9 Pro](https://www.nameless.wiki/getting-started/flash-instructions/oneplus/lemonadep) 作为主要参考，由于 Android 15 版镜像包与官网上 (Android 14) 的不太一样，因此 `fastboot flash` 部分命令也参考了 [Nameless CLO | OnePlus 11](https://www.nameless.wiki/getting-started/flash-instructions/oneplus/salami)
- Android 大佬 yezhiyi9670 的博客 [\[搞机\] 一加（OnePlus）手机刷机 Root](https://ak-ioi.com/1003-oneplus-flash-root) 帮我解决了 `fastboot devices` 检测不到设备的问题，另外也学到了很多了刷机知识，甚至这篇博客的完善也很大部分参考自 yezhiyi9670 大佬的博客
- 浅笑一夏 up 主的视频 [一加9、一加9pro的twrp刷入及root和刷机教程](https://www.bilibili.com/video/BV18o4y1D7hP) 以及相应的博客文章 [oneplus 9 pro 一加9pro第三方系统刷机包及教程集合+救砖教程](https://yuanfangblog.xyz/technology/425.html) 是最开始找的资料，带我了解到了刷机的大致流程，虽然我实际刷机时遇到了很多 UP 主没有遇到的错误，尤其是在功能太过强大的 TWRP 中遇到报错导致我一番乱搞差点把手机刷成了砖呜呜
- HikariLan 大佬的博客 [基于 HyperOS 2.0 和 Android 15 的个人 Magisk + Xposed 模块分享 - HikariLan's Blog](https://blog.hikarilan.life/thinking/1909/%E5%9F%BA%E4%BA%8E-hyperos-2-0-%E5%92%8C-android-15-%E7%9A%84%E4%B8%AA%E4%BA%BA-magisk-xposed-%E6%A8%A1%E5%9D%97%E5%88%86%E4%BA%AB/) 让我知道了 Magisk 下适用于 Android 15 的 Lsposed 模块，以及很多实用的模块和软件
- 还有当火龙 888 发怒时及时为我安抚机魂的 [琪露诺老师](https://zh.touhouwiki.net/index.php?title=%E7%90%AA%E9%9C%B2%E8%AF%BA&variant=zh)，琪老师成功让我的寝室免于 N 场可怕的火灾，是真正的圣 Baka !
    - 在琪老师的强烈要求下，仔细观察可以看到本文 OnePlus 9 Pro (火龙 888) 的几乎每一张照片中都映出了琪老师神圣的影子 (ᗜ‿ᗜ)
- 最后感谢强大的 [D 老师](https://www.deepseek.com/), 刷机方案最后敲定是在 D 老师的淳淳教导下完成的，并且解决了不少我在实际刷机过程中的疑问

[^1]: 参考自: [确认如何从关机状态进入 Fastboot](https://ak-ioi.com/1003-oneplus-flash-root/#toc-head-11)

[^2]: 参考自: [\[Q\] ADB Can't Read Sideload](https://xdaforums.com/t/q-adb-cant-read-sideload.1990405/post-38001753)

[^3]: 参考自：[手机游玩 galgame 模拟器的终极解决方案: winlator](https://galgame.dev/topic/37/%E6%89%8B%E6%9C%BA%E6%B8%B8%E7%8E%A9-galgame-%E6%A8%A1%E6%8B%9F%E5%99%A8%E7%9A%84%E7%BB%88%E6%9E%81%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88-winlator)