---
layout: post
title:  "automake"
date:   2019-08-21 05:30:30 +0800
categories: [HW]
excerpt: GNU automake.
tags:
  - Application
---

![](/assets/PDB/BiscuitOS/kernel/IND00000Q.jpg)

> Email: BuddyZhang1 <buddy.zhang@aliyun.com>

## 目录

> - [automake 简介](#A00)
>
> - [实践准备](#B00)
>
>   - [硬件准备](#B000)
>
>   - [软件准备](#B001)
>
> - [automake 部署](#C00)
>
>   - [BiscuitOS 部署](#C0000)
>
>   - [工程实践部署](#C0001)
>
> - [automake 使用](#D00)
>
> - [automake 测试](#E00)
>
> - [automake 进阶研究](#F0)
>
> - [附录/捐赠](#Donate)

------------------------------------------

<span id="A00"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000G.jpg)

## automake 简介

GNU automake 是从 Makefile.am (定义一系列 make 变量的文件) 自动生成
Makefile.in 文件的工具。生成的 Makefile.in 符合 GNU Makefile 标准。
因为不同的平台需要修改 Makefile，所以编写 configure 脚本自动修改 Makefile，
再执行 ./configure && make && make install 命令安装包，形成了 GNU 构建系
统。目前 BiscuitOS 已经支持 GNU automake 的移植和实践。开发者
可用通过下面的章节进行 GNU automake 的使用。

------------------------------------------

<span id="B00"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000Y.jpg)

## 实践准备

> - [硬件准备](#B000)
>
> - [软件准备](#B001)

------------------------------------------

<span id="B000"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000R.jpg)

## 硬件准备

BiscuitOS 对 automake 的实践分别提供了纯软件实践平台和硬件实践
平台，如果开发者需要在硬件平台上实践，那么需要准备本节提到的内容。

> - [硬件平台](#B0000)
>
> - [逻辑分析仪](#B0001)
>
> - [示波器](#B0002)

---------------------------------------------------

##### <span id="B0000">硬件平台</span>

开发者需要准备一块 RaspberryPi 4B 开发板，并配有 SD 卡，SD 读卡器，
TTL 转 USB 串口一个。

> - [RaspberryPi 官网](https://www.raspberrypi.org/)

![](/assets/PDB/RPI/RPI000016.png)

------------------------------------------

##### <span id="B0001">逻辑分析仪</span>

逻辑分析仪能够帮助开发者快速分析数据，测试硬件信号功能，稳定性，
大量数据采样等。逻辑分析仪不是必须的，这里推荐使用 DreamSourceLab
开发的 DSLogic:

![](/assets/PDB/RPI/RPI000012.jpg)

DSLogic 逻辑分析仪数据工具：

![](/assets/PDB/RPI/RPI000062.png)

![](/assets/PDB/RPI/RPI000063.png)

![](/assets/PDB/RPI/RPI000066.png)

> - [DreamSourceLab: DSLogic 官网](https://dreamsourcelab.cn/product/dslogic-plus/)

----------------------------------------

##### <span id="B0002">示波器</span>

示波器能够帮助开发者对硬件总线进行最透彻的分析，示波器测量
的数据具有可靠性高，精度高的特定，是分析 I2C 问题不可或缺的
工具。示波器建议准备，这里推荐使用 DreamSourceLab 开发的 DsCope:

![](/assets/PDB/RPI/RPI000013.jpg)

DSCope 示波器采用样图:

![](/assets/PDB/RPI/RPI000006.png)

![](/assets/PDB/RPI/RPI000057.png)

![](/assets/PDB/RPI/RPI000069.png)

> - [DreamSourceLab: DSLogic 官网](https://dreamsourcelab.cn/product/dscope-u2p20/)

------------------------------------------

<span id="B001"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000K.jpg)

## 软件准备

在进行 automake 开发之前，开发者应该准备 BiscuitOS 的开发
环境，开发者应该根据不同的需求进行准备，如下:

> - [BiscuitOS Qemu 软件方案开发环境部署](https://biscuitos.github.io/blog/Kernel_Build/#Linux_5X)
>
> - [BiscuitOS RaspberryPi 硬件方案开发环境部署](https://biscuitos.github.io/blog/BiscuitOS_Catalogue/#RaspberryPi)

------------------------------------------

<span id="C00"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000L.jpg)

## automake 部署

automake 可以在 BiscuitOS 上实践，也可以在实际的工程实践
中使用，开发者可以参考下面的目录进行使用:

> - [BiscuitOS 部署](#C0000)
>
> - [工程实践部署](#C0001)

------------------------------------------

<span id="C0000"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000Q.jpg)

## BiscuitOS 部署

BiscuitOS 以及完整支持 automake，并基于 Kbuild 编译系统，制作了一套
便捷的 automake 开发环境，开发者可以参考如下步骤进行快速开发。

> - [automake 源码获取](#C00000)
>
> - [automake 源码编译](#C00001)
>
> - [automake 应用安装](#C00002)

--------------------------------------------

#### <span id="C00000">automake 源码获取</span>

开发者在准备好了 BiscuitOS 开发环境之后，只需按照下面步骤就可以
便捷获得 automake 开发所需的源码及环境, 以 RaspberryPi 4B 硬件
开发环境为例，其他 Linux 版本类似，开发者自行进行修改:

{% highlight bash %}
cd BiscuitOS
make RaspberryPi_4B_defconfig
make menuconfig
{% endhighlight %}

![](/assets/PDB/RPI/RPI000038.png)

选择 "Package --->" 并进入下一级菜单

![](/assets/PDB/RPI/RPIALL.png)

设置 "automake --->" 为 "Y"。设置完毕之后，
保存并退出.

-----------------------------------------------

#### <span id="C00001">automake 源码编译</span>

automake 的编译很简单，只需执行如下命令就可以快速编
译 (以 RaspberryPi 4B 为例):

{% highlight bash %}
cd BiscuitOS/
make
cd BiscuitOS/output/RaspberryPi_4B/package/automake-x.x.x/
make prepare
make download
make
make install
make pack
{% endhighlight %}

---------------------------------------------------

#### <span id="C00002">automake 应用安装</span>

开发者由于采用了 QEMU 方案或者硬件 RaspberryPi 方案进行实践，
应用更新采用不同的方式，两种方案的更新方式如下:

开发者如果使用 QEMU 方案，那么只需执行如下命令就能
进行软件更新:

{% highlight bash %}
cd BiscuitOS/
cd BiscuitOS/output/RaspberryPi_4B/package/automake-x.x.x/
make install
make pack
{% endhighlight %}

通过上面的命令就可以将软件更新到系统了，接下来就是运行 BiscuitOS.

开发者如果使用是硬件 RaspberryPi 方案，更新的方式多种方式，
可以参考下面方式:

> - [BiscuitOS NFS 方式更新](https://biscuitos.github.io/blog/RaspberryPi_4B-Usermanual/#A020140)
>
> - [BiscuitOS SD 卡方式更新](https://biscuitos.github.io/blog/RaspberryPi_4B-Usermanual/#A020141)

------------------------------------------

<span id="C0001"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000U.jpg)

## 工程实践部署

开发者也可以将 automake 部署到实际的工程项目中，开发者
可以根据 BiscuitOS 中部署的方法加入到工程实践中，请参考
如下章节:

> - [BiscuitOS automake 部署](#C0000)

接着使用如下命令 (以 RaspberryPi 4B 为例):

{% highlight bash %}
cd BiscuitOS/
cd BiscuitOS/output/RaspberryPi_4B/package/automake-x.x.x/
make download
{% endhighlight %}

在该目录下，README 和 Makefile 文档介绍了 automake 的
使用方法，以及 Makefile 编译方法，开发者可以参考以上内容
进行工程部署。

------------------------------------------

<span id="D00"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000D.jpg)

## automake 使用

automake 安装完毕之后，启动 BiscuitOS，在 BiscuitOS 上
使用 automake 如下:

![](/assets/PDB/RPI/RPI000106.png)

------------------------------------------

<span id="E00"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000E.jpg)

## automake 测试

等待更新

------------------------------------------

<span id="F00"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000F.jpg)

## automake 进阶研究

等待更新

------------------------------------------

<span id="Donate"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000K.jpg)

## 附录

> [BiscuitOS Home](https://biscuitos.github.io/)
>
> [BiscuitOS Catalogue](https://biscuitos.github.io/blog/BiscuitOS_Catalogue/)
>
> [BiscuitOS Kernel Build](https://biscuitos.github.io/blog/Kernel_Build/)
>
> [Linux Kernel](https://www.kernel.org/)
>
> [Bootlin: Elixir Cross Referencer](https://elixir.bootlin.com/linux/latest/source)

## 捐赠一下吧 🙂

![MMU](/assets/PDB/BiscuitOS/kernel/HAB000036.jpg)
