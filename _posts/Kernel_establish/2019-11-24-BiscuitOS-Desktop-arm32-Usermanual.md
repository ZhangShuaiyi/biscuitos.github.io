---
layout: post
title:  "BiscuitOS-Desktop arm32 Usermanual"
date:   2019-11-24 14:45:30 +0800
categories: [Build]
excerpt: BiscuitOS-Desktop arm32 Usermanual.
tags:
  - Linux
---

![](/assets/PDB/BiscuitOS/kernel/IND00000L0.PNG)

![](/assets/PDB/RPI/RPI100100.png)

## 目录

> - [开发环境部署](#A)
>
>   - [硬件准备](#A0)
>
>   - [软件准备](#A1)
>
> - [内核部署](#C)
>
>   - [BiscuitOS-Desktop 项目部署](#C0)
>
>   - [内核配置](#C1)
>
>   - [内核编译](#C2)
>
> - [BiscuitOS 使用](#D)
>
>   - [BiscuitOS 安装](#D0)
>
>   - [BiscuitOS 运行](#D1)
>
> - [驱动部署](#E)
>
>   - [BiscuitOS 驱动开发](#E0)
>
>   - [通用驱动开发](#E1)
>
>   - [驱动实践](#E2)
>
> - [应用程序部署](#F)
>
>   - [BiscuitOS 部署](#F0)
>
>   - [游戏部署](#F2)
>
> - [调试部署](#G)
>
> - [附录/捐赠](#Z)

------------------------------------------

<span id="A"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000L0.PNG)

## 开发环境部署

> - [项目简介](#A2)
>
> - [硬件准备](#A0)
>
> - [软件准备](#A1)

-----------------------------------------------

<span id="A2"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000S.jpg)

## 项目简介

BiscuitOS 项目是一个用于制作 Linux 0.x、1.x、2.x、3.x、4.x、5.0
通用精简操作系统，其目标是为开发者提供一套纯软件的 Qemu 实践平台
或者硬件 RaspberryPi 实践平台，让开发者能够便捷、简单、快速的
在各个版本上实践 Linux。BiscuitOS-Destop 项目是基于 BiscuitOS
构建的桌面操作系统，开发者可以使用该项目从源码开始构建并定制
属于自己的桌面操作系统。更多 BiscuitOS 信息请范围下列网站:

> - [BiscuitOS 主页](https://biscuitos.github.io/)
>
> - [BiscuitOS 博客](https://biscuitos.github.io/blog/BiscuitOS_Catalogue/)

-----------------------------------------------

<span id="A0"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000M.jpg)

## 硬件准备

![](/assets/PDB/RPI/RPI000046.JPG)

由于项目构建基于 Ubuntu，因此需要准备一台运行
Ubuntu 14.04/16.04/18.04 的主机，主机需要保持网络的连通。

-----------------------------------------------

<span id="A1"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000K.jpg)

## 软件准备

BiscuitOS 制作的系统都是由源码编译而来，这个开发者带来
更多有趣的可能，开发者在使用 BiscuitOS 构建 BiscuitOS-Desktop
的系统之前，需要做好如下准备:

> - [基础软件安装](#A10)
>
> - [BiscuitOS 部署](#A11)

---------------------------------------------

#### <span id="A10">基础软件安装</span>

开发者首先准备一台 Linux 发行版电脑，推荐 Ubuntu 16.04/Ubuntu 18.04,
Ubuntu 电脑的安装可以上网查找相应的教程。准备好相应的开发主机之后，
接下来是安装运行 BiscuitOS 项目所需的基础开发工具。以 Ubuntu 为例
安装基础的开发工具。开发者可以按如下文档进行安装 (必须):

> - [BiscuitOS 基础开发工具安装指南](https://biscuitos.github.io/blog/Develop_tools)

----------------------------------------

#### <span id="A11">BiscuitOS 部署</span>

基础环境搭建完毕之后，开发者从 GitHub 上获取 BiscuitOS 项目源码，
使用如下命令：

{% highlight bash %}
git clone https://github.com/BiscuitOS/BiscuitOS.git --depth=1
cd BiscuitOS
{% endhighlight %}

至此，BiscuitOS 已经部署完毕.
BiscuitOS 目前已经支持 BiscuitOS-Desktop 的开发部署，开发者在
部署完 BiscuitOS 环境之后，可以参考下面命令进行部署:

{% highlight bash %}
cd BiscuitOS
make BiscuitOS-Desktop_defconfig 
make
cd BiscuitOS/output/BiscuitOS-Desktop
{% endhighlight %}

执行上面的命令之后，BiscuitOS 项目就会自动部署 "BiscuitOS-Desktop"
的开发环境，并自动生成各个模块编译规则，开发者请自行参考，例如:

{% highlight perl %}
 ____  _                _ _    ___  ____  
| __ )(_)___  ___ _   _(_) |_ / _ \/ ___|
|  _ \| / __|/ __| | | | | __| | | \___ \ 
| |_) | \__ \ (__| |_| | | |_| |_| |___) |
|____/|_|___/\___|\__,_|_|\__|\___/|____/ 

***********************************************
Output:
 BiscuitOS/output/BiscuitOS-Desktop

linux:
 BiscuitOS/output/BiscuitOS-Desktop/linux/linux 

README:
 BiscuitOS/output/BiscuitOS-Desktop/README.md 

***********************************************
{% endhighlight %}

在上面的输出信息中，指出了 "BiscuitOS-Desktop" 项目
的目录位置，以及 linux 源码目录，更重要的是各个模块
编译规则，开发者应该重点参考 README.md 的内容.

------------------------------------------

<span id="C"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000P.jpg)

## 内核部署

BiscuitOS 项目的目标就是为开发者提供一套快速实践内核的平台，
因此 BiscuitOS 支持完整的内核开发，开发者请参考下列步骤进行开发:

> - [BiscuitOS-Desktop 项目部署](#C0)
>
> - [内核配置](#C1)
>
> - [内核编译](#C2)

------------------------------------------

<span id="C0"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000O.jpg)

## BiscuitOS-Desktop 项目部署

BiscuitOS 目前已经支持 BiscuitOS-Desktop 的开发部署，开发者在
部署完 BiscuitOS 环境之后，可以参考下面命令进行部署:

{% highlight bash %}
cd BiscuitOS
make BiscuitOS-Desktop_defconfig
make
cd BiscuitOS/output/BiscuitOS-Desktop
{% endhighlight %}

执行上面的命令之后，BiscuitOS 项目就会自动部署 "BiscuitOS-Desktop"
的开发环境，并自动生成各个模块编译规则，开发者请自行参考，例如:

{% highlight perl %}
 ____  _                _ _    ___  ____  
| __ )(_)___  ___ _   _(_) |_ / _ \/ ___|
|  _ \| / __|/ __| | | | | __| | | \___ \ 
| |_) | \__ \ (__| |_| | | |_| |_| |___) |
|____/|_|___/\___|\__,_|_|\__|\___/|____/ 

***********************************************
Output:
 BiscuitOS/output/BiscuitOS-Desktop/

linux:
 BiscuitOS/output/BiscuitOS-Desktop/linux/linux 

README:
 BiscuitOS/output/BiscuitOS-Desktop/README.md 

***********************************************
{% endhighlight %}

在上面的输出信息中，指出了 "BiscuitOS-Desktop" 项目
的目录位置，以及 linux 源码目录，更重要的是各个模块
编译规则，开发者应该重点参考 README.md 的内容.

------------------------------------------

<span id="C1"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000Q.jpg)

## 内核配置

对于内核配置，根据 README.md 的提示，配置内核可以
参考如下命令:

{% highlight bash %}
cd BiscuitOS/output/BiscuitOS-Desktop/linux/linux
make ARCH=arm clean
make ARCH=arm vexpress_defconfig
make ARCH=arm menuconfig
{% endhighlight %}

------------------------------------------

<span id="C2"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000Y.jpg)

## 内核编译

内核配置完毕之后，接下来就是编译内核，根据 README.md 里提供的
命令进行编译，具体命令如下:

{% highlight bash %}
make ARCH=arm CROSS_COMPILE=BiscuitOS/output/BiscuitOS-Desktop/arm-linux-gnueabi/arm-linux-gnueabi/bin/arm-linux-gnueabi- -j8
make ARCH=arm CROSS_COMPILE=BiscuitOS/output/BiscuitOS-Desktop/arm-linux-gnueabi/arm-linux-gnueabi/bin/arm-linux-gnueabi- dtbs
{% endhighlight %}

![LINUXP](/assets/PDB/BiscuitOS/kernel/BUDX000335.png)

------------------------------------------

<span id="D"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000F.jpg)

## BiscuitOS 使用

在准备完各个模块之后，开发者完成本节的内容之后，就可以运行一个
完整的 BiscuitOS 系统。开发者请参考如下内容:

> - [BiscuitOS 安装](#D0)
>
> - [BiscuitOS 运行](#D1)

----------------------------------------

## <span id="D0">BiscuitOS 安装</span>

BiscuitOS 系统镜像的安装制作很简单，请参考如下命令:

{% highlight bash %}
cd BiscuitOS/output/BiscuitOS-Desktop
./RunQemuKernel.sh pack
{% endhighlight %}

运行上面的命令之后，会生成名为 "BiscuitOS-Desktop.img" 的文件，
该文件就是 BiscuitOS 系统镜像。

----------------------------------------

## <span id="D1">BiscuitOS 运行</span>

BiscuitOS 的运行很简单，开发者执行使用如下命令即可:

{% highlight bash %}
cd BiscuitOS/output/BiscuitOS-Desktop
./RunQemuKernel.sh
{% endhighlight %}

如果需要使用网络功能，请使用如下命令:

{% highlight bash %}
cd BiscuitOS/output/BiscuitOS-Desktop
./RunQemuKernel.sh net
{% endhighlight %}

![](/assets/PDB/RPI/RPI000300.png)

默认账号 "biscuitos", 默认密码 "root"

![](/assets/PDB/RPI/RPI000301.png)

开发者可以在 BiscuitOS-Desktop 中使用鼠标，当要从恢复
鼠标可以使用 "Ctrl + Alt + G".

------------------------------------------

<span id="E"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000M.jpg)

## 驱动部署

BiscuitOS 目前已经完整支持驱动的开发，开发者可以使用 BiscuitOS
提供的驱动开发机制进行驱动开发，也可以使用通用的驱动开发办法
在 BiscuitOS 系统上调试驱动，开发者可以参考如下文档:

> - [BiscuitOS 驱动开发](#E0)
>
> - [通用驱动开发](#E1)
>
> - [驱动实践](#E2)

------------------------------------------

<span id="E0"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000P.jpg)

## BiscuitOS 驱动开发

BiscuitOS 以及完整支持驱动开发体系，并基于 Kbuild 编译系统，制作了一套
便捷的驱动开发环境，开发者可以参考如下步骤进行快速开发，本次以一个 Platform
驱动为例进行讲解。

> - [Platform 源码获取](#E00)
>
> - [Platform 源码编译](#E01)
>
> - [Platform 模块安装](#E02)
>
> - [Platform 模块使用](#E03)

--------------------------------------------

#### <span id="E00">Platform 源码获取</span>

首先开发者应该准备基于 BiscuitOS-Desktop arm32 开发环境，然后
使用如下命令获得 Platform 所需的开发环境：

{% highlight bash %}
cd BiscuitOS
make BiscuitOS-Desktop_defconfig
make menuconfig
{% endhighlight %}

![](/assets/PDB/RPI/RPI000038.png)

选择 "Package --->" 并进入下一级菜单

![](/assets/PDB/RPI/RPI000304.png)

选择 "Platform: Device Driver and Application"，并进入下一级菜单

![](/assets/PDB/RPI/RPI000305.png)

设置 "platform Core Module --->" 为 "Y"。设置完毕之后，
保存并退出.

-----------------------------------------------

#### <span id="E01">Platform 源码编译</span>

Platform 的编译很简单，只需执行如下命令就可以快速编译:

{% highlight bash %}
make
cd BiscuitOS/output/BiscuitOS-Desktop/package/platform_core_module-0.0.1/
make prepare
make download
make
make install
make pack
{% endhighlight %}

-----------------------------------------------

#### <span id="E02">Platform 模块安装</span>

驱动的安装特别简单，使用如下命令:

{% highlight bash %}
cd BiscuitOS/output/BiscuitOS-Desktop/package/platform_core_module-0.0.1/
make install
make pack
{% endhighlight %}

-----------------------------------------------

#### <span id="E03">Platform 模块使用</span>

模块的使用也很简单，使用如下命令:

{% highlight bash %}
cd BiscuitOS/output/BiscuitOS-Desktop/
./RunBiscuitOS.sh
{% endhighlight %}

运行 BiscuitOS 之后，在 BiscuitOS 上使用如下命令:

{% highlight bash %}
cd lib/modules/5.0.0/extra/
insmod platform_core_module-0.0.1
{% endhighlight %}

------------------------------------------

<span id="E1"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000W.jpg)

## 通用驱动开发

通用驱动开发模式就是按标准的内核驱动开发方式，将驱动放到内核
源码树内或者源码树之外进行开发，这里以内核源码树内开发为例:

> - [驱动源码](#E10)
>
> - [驱动安置](#E11)
>
> - [驱动配置](#E12)
>
> - [驱动编译](#E13)
>
> - [驱动安装](#E14)
>
> - [驱动使用](#E15)

------------------------------------------

<span id="E10"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000J.jpg)

## 驱动源码

开发者首先准备一份驱动源码，可以操作如下源码，本节中使用一份 
misc 驱动，并命名为 BiscuitOS_drv.c，具体源码如下：

![](/assets/PDB/RPI/RPI000056.PNG)

------------------------------------------

<span id="E11"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000L.jpg)

## 驱动安置

开发者准备好源码之后，接着将源码添加到内核源码树。开发者在内核源码树
目录下，找到 drivers 目录，然后在 drivers 目录下创建一个名为 BiscuitOS
的目录，然后将源码 BiscuitOS_drv.c 放入到 BiscuitOS 目录下。接着添加
Kconfig 和 Makefile 文件，具体内容如下：

##### **Kconfig**

{% highlight bash %}
menuconfig BISCUITOS_DRV
    bool "BiscuitOS Driver"

if BISCUITOS_DRV

config BISCUITOS_MISC
    bool "BiscuitOS misc driver"

endif # BISCUITOS_DRV
{% endhighlight %}

##### **Makefile**

{% highlight bash %}
obj-$(CONFIG_BISCUITOS_MISC)  += BiscuitOS_drv.o
{% endhighlight %}

接着修改内核源码树 drivers 目录下的 Kconfig，在文件的适当位置加上如下代码：

{% highlight bash %}
source "drivers/BiscuitOS/Kconfig"
{% endhighlight %}

然后修改内核源码树 drivers 目录下的 Makefile，在文件的末尾上加上如下代码：

{% highlight bash %}
obj-$(CONFIG_BISCUITOS_DRV)  += BiscuitOS/
{% endhighlight %}

------------------------------------------

<span id="E12"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000Z.jpg)

## 驱动配置

准备好所需的文件之后，接下来进行驱动在内核源码树中的配置，使用如下命令:

{% highlight bash %}
cd BiscuitOS/output/BiscuitOS-Desktop/linux/linux
make ARCH=arm menuconfig
{% endhighlight %}

![LINUXP](/assets/PDB/BiscuitOS/kernel/BUDX000337.png)

首先在目录中找到 **Device Driver --->** 回车并进入其中。

![LINUXP](/assets/PDB/BiscuitOS/kernel/BUDX000338.png)

接着在目录中找到 **BiscuitOS Driver --->** 按 Y 选中并按回车键进入。

![LINUXP](/assets/PDB/BiscuitOS/kernel/BUDX000339.png)

最后按 Y 键选中 **BiscuitOS mis driver**，保存并退出内核配置

------------------------------------------

<span id="E13"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000Z.jpg)

## 驱动编译

配置完驱动之后，接下来将编译驱动。开发者使用如下命令编译驱动:

{% highlight bash %}
make ARCH=arm CROSS_COMPILE=BiscuitOS/output/BiscuitOS-Desktop/arm-linux-gnueabi/arm-linux-gnueabi/bin/arm-linux-gnueabi- modules -j4
make ARCH=arm INSTALL_MOD_PATH=BiscuitOS/output/BiscuitOS-Desktop/rootfs/rootfs/ modules_install
{% endhighlight %}

从编译的 log 可以看出 BiscuitOS_drv.c 已经被编译进内核。

![LINUXP](/assets/PDB/BiscuitOS/kernel/BUDX000100.jpg)

------------------------------------------

<span id="E14"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000X.jpg)

## 驱动安装

驱动安装制作很简单，请参考如下命令:

{% highlight bash %}
cd BiscuitOS/output/BiscuitOS-Desktop
./RunQemuKernel.sh pack
{% endhighlight %}

-----------------------------------------------

#### <span id="E15">驱动使用</span>

模块的使用也很简单，使用如下命令:

{% highlight bash %}
cd BiscuitOS/output/BiscuitOS-Desktop/
./RunBiscuitOS.sh
{% endhighlight %}

运行 BiscuitOS 之后，在 BiscuitOS 上使用如下命令:

{% highlight bash %}
cd lib/modules/5.0.0/kernel/driver/BiscuitOS/
insmod misc.ko
{% endhighlight %}

------------------------------------------

<span id="E2"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000Z.jpg)

## 驱动实践

BiscuitOS 提供了丰富的驱动开发教程，开发者可以参考如下文档:

> - [BiscuitOS 驱动实践合集](https://biscuitos.github.io/blog/BiscuitOS_Catalogue/#Enginerring)
> 
> - [BiscuitOS RaspberryPi 驱动合集](https://biscuitos.github.io/blog/BiscuitOS_Catalogue/#RaspberryPi)

------------------------------------------

<span id="F"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000F.jpg)

## 应用程序部署

BiscuitOS 目前提供了一套完整的应用程序开发系统，开发者可以
使用这套系统快速实践应用程序，开发者也可以使用传统的方法
在 BiscuitOS 上实践应用程序:

> - [BiscuitOS 部署](#F0)
>
> - [游戏部署](#F1)

----------------------------------

## <span id="F0">BiscuitOS 部署</span>

BiscuitOS 提供了多种动态库、静态块、GNU 工具、BiscuitOS 应用程序，
以及各种开源项目，开发者可以参考下文进行使用:

> - [BiscuitOS 上使用 GNU hello 项目](https://biscuitos.github.io/blog/USER_hello/)
>
> - [BiscuitOS 支持应用列表](https://biscuitos.github.io/blog/APP-Usermanual/)

----------------------------------

## <span id="F1">游戏部署</span>

BiscuitOS 也支持游戏，开发者可以参考如下文章，为自己的
开发之旅带来更多的快乐:

> - [Snake 贪吃蛇](https://biscuitos.github.io/blog/USER_snake/)
>
> - [2048](https://biscuitos.github.io/blog/USER_2048/)
>
> - [tetris 俄罗斯方块](https://biscuitos.github.io/blog/USER_tetris/)

------------------------------------------

<span id="G"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000T.jpg)

## 调试部署

BiscuitOS 的 Linux 内核也支持多种调试，其中比较重
要的方式就是 GDB 调试，开发者可以使用 GDB 对内核进
行单步调试，具体步骤如下：(具体参照 BiscuitOS 中的 README)

首先使用 QEMU 的调试工具，将内核挂起等待调试，使用如下命令:

{% highlight bash %}
cd BiscuitOS/output/BiscuitOS-Desktop
./RunQemuKernel.sh debug
{% endhighlight %}

![LINUXP](/assets/PDB/BiscuitOS/kernel/BUDX000315.png)

接着在另外一个终端中输入如下命令，作为 gdb server

{% highlight bash %}
BiscuitOS/output/BiscuitOS-Desktop/arm-linux-gnueabi/arm-linux-gnueabi/bin/arm-linux-gnueabi-gdb BiscuitOS/output/BiscuitOS-Desktop/linux/linux/vmlinux
{% endhighlight %}

输入以上命令之后，在第二个终端中，会进入 gdb 模式，此时以此输入一下命名进行
gdb 挂载：

{% highlight bash %}
(gdb) target remote :1234
{% endhighlight %}

然后开发者可以根据自己的需求进行内核调试，比如在 start_kernel 出添加
一个断点调试，如下:

{% highlight bash %}
(gdb) b start_kernel
(gdb) c
(gdb) list
(gdb) info reg
{% endhighlight %}

![LINUXP](/assets/PDB/BiscuitOS/kernel/BUDX000316.png)

更多内核调试，请查考文档:

> - [BiscuitOS 调试](https://biscuitos.github.io/blog/BiscuitOS_Catalogue/#Debug)

-----------------------------------------------

# <span id="Z">附录</span>

> [BiscuitOS Home](https://biscuitos.github.io/)
>
> [BiscuitOS Catalogue](https://biscuitos.github.io/blog/BiscuitOS_Catalogue/)
>
> [Linux Kernel](https://www.kernel.org/)
>
> [Bootlin: Elixir Cross Referencer](https://elixir.bootlin.com/linux/latest/source)

## 捐赠一下吧 🙂

![MMU](/assets/PDB/BiscuitOS/kernel/HAB000036.jpg)
