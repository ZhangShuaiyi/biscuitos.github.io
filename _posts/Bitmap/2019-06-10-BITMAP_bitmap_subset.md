---
layout: post
title:  "bitmap_subset"
date:   2019-06-10 05:30:30 +0800
categories: [HW]
excerpt: BITMAP bitmap_subset().
tags:
  - Tree
---

![DTS](/assets/PDB/BiscuitOS/kernel/IND00000B.jpg)

> [Github: bitmap_subset](https://github.com/BiscuitOS/HardStack/tree/master/Algorithem/bitmap/API/bitmap_subset)
>
> Email: BuddyZhang1 <buddy.zhang@aliyun.com>

# 目录

> - [源码分析](#源码分析)
>
> - [实践](#实践)
>
> - [附录](#附录)

-----------------------------------

# <span id="源码分析">源码分析</span>

{% highlight bash %}
static inline int bitmap_subset(const unsigned long *src1,
                        const unsigned long *src2, unsigned int nbits)
{
        if (small_const_nbits(nbits))
                return ! ((*src1 & ~(*src2)) & BITMAP_LAST_WORD_MASK(nbits));
        else
                return __bitmap_subset(src1, src2, nbits);
}
{% endhighlight %}

bitmap_subset() 函数用于判断 src2 指定的位数内是不是 src1 ‘1’ 的子集。参数 src1
指向第一个 bitmap；参数 src2 指向第二个 bitmap；nbits 参数指明检查的 bit 数。
函数首先通过 small_const_nbits() 函数判断 nbits 是否符合 BITS_PER_LONG 的常数
要求，如果满足，则将 src1 与 src2 的反码进行相与，相与的结果与上
BITMAP_LAST_WORD_MASK() 生成的掩码，以此确定是否属于 src1 的子集；如果不符合
上述掉件，那么函数直接调用 __bitmap_subset() 函数进行检查。

> - [BITMAP_LAST_WORD_MASK](https://biscuitos.github.io/blog/BITMAP_BITMAP_LAST_WORD_MASK)
>
> - [\_\_bitmap_subset](https://biscuitos.github.io/blog/BITMAP___bitmap_subset)

--------------------------------------------------

# <span id="实践">实践</span>

> - [驱动源码](#驱动源码)
>
> - [驱动安装](#驱动安装)
>
> - [驱动配置](#驱动配置)
>
> - [驱动编译](#驱动编译)
>
> - [驱动运行](#驱动运行)
>
> - [驱动分析](#驱动分析)

#### <span id="驱动源码">驱动源码</span>

{% highlight c %}
/*
 * Bitmap.
 *
 * (C) 2019.06.10 <buddy.zhang@aliyun.com>
 *
 * This program is free software; you can redistribute it and/or modify
 * it under the terms of the GNU General Public License version 2 as
 * published by the Free Software Foundation.
 */

#include <linux/kernel.h>
#include <linux/init.h>

/* header of bitmap */
#include <linux/bitmap.h>

static __init int bitmap_demo_init(void)
{
	unsigned long bitmap0 = 0xff45;
	unsigned long bitmap1 = 0x7845;

	/* Check bitmap whether '1' subset */
	if (bitmap_subset(&bitmap0, &bitmap1, 4))
		printk("%#lx '1' subset %#lx through 0 to 3.\n",
						bitmap0, bitmap1);

	return 0;
}
device_initcall(bitmap_demo_init);
{% endhighlight %}

#### <span id="驱动安装">驱动安装</span>

驱动的安装很简单，首先将驱动放到 drivers/BiscuitOS/ 目录下，命名为 bitmap.c，
然后修改 Kconfig 文件，添加内容参考如下：

{% highlight bash %}
diff --git a/drivers/BiscuitOS/Kconfig b/drivers/BiscuitOS/Kconfig
index 4edc5a5..1a9abee 100644
--- a/drivers/BiscuitOS/Kconfig
+++ b/drivers/BiscuitOS/Kconfig
@@ -6,4 +6,14 @@ if BISCUITOS_DRV
config BISCUITOS_MISC
        bool "BiscuitOS misc driver"
+config BISCUITOS_BITMAP
+       bool "bitmap"
+
+if BISCUITOS_BITMAP
+
+config DEBUG_BISCUITOS_BITMAP
+       bool "bitmap_subset"
+
+endif # BISCUITOS_BITMAP
+
endif # BISCUITOS_DRV
{% endhighlight %}

接着修改 Makefile，请参考如下修改：

{% highlight bash %}
diff --git a/drivers/BiscuitOS/Makefile b/drivers/BiscuitOS/Makefile
index 82004c9..9909149 100644
--- a/drivers/BiscuitOS/Makefile
+++ b/drivers/BiscuitOS/Makefile
@@ -1 +1,2 @@
obj-$(CONFIG_BISCUITOS_MISC)     += BiscuitOS_drv.o
+obj-$(CONFIG_BISCUITOS_BITMAP)     += bitmap.o
--
{% endhighlight %}

#### <span id="驱动配置">驱动配置</span>

驱动配置请参考下面文章中关于驱动配置一节。在配置中，勾选如下选项，如下：

{% highlight bash %}
Device Driver--->
    [*]BiscuitOS Driver--->
        [*]bitmap
            [*]bitmap_subset()
{% endhighlight %}

具体过程请参考：

> [Linux 5.0 开发环境搭建 -- 驱动配置](https://biscuitos.github.io/blog/Linux-5.0-arm32-Usermanual/#%E9%A9%B1%E5%8A%A8%E9%85%8D%E7%BD%AE)

#### <span id="驱动编译">驱动编译</span>

驱动编译也请参考下面文章关于驱动编译一节：

> [Linux 5.0 开发环境搭建 -- 驱动编译](https://biscuitos.github.io/blog/Linux-5.0-arm32-Usermanual/#%E7%BC%96%E8%AF%91%E9%A9%B1%E5%8A%A8)

#### <span id="驱动运行">驱动运行</span>

驱动的运行，请参考下面文章中关于驱动运行一节：

> [Linux 5.0 开发环境搭建 -- 驱动运行](https://biscuitos.github.io/blog/Linux-5.0-arm32-Usermanual/#%E9%A9%B1%E5%8A%A8%E8%BF%90%E8%A1%8C)

启动内核，并打印如下信息：

{% highlight bash %}
usbcore: registered new interface driver usbhid
usbhid: USB HID core driver
0xff45 '1' subset 0x7845 through 0 to 3.
aaci-pl041 10004000.aaci: ARM AC'97 Interface PL041 rev0 at 0x10004000, irq 24
aaci-pl041 10004000.aaci: FIFO 512 entries
oprofile: using arm/armv7-ca9
{% endhighlight %}

#### <span id="驱动分析">驱动分析</span>

bitmap 子集检测

-----------------------------------------------

# <span id="附录">附录</span>

> [BiscuitOS Home](https://biscuitos.github.io/)
>
> [BiscuitOS Driver](https://biscuitos.github.io/blog/BiscuitOS_Catalogue/)
>
> [BiscuitOS Kernel Build](https://biscuitos.github.io/blog/Kernel_Build/)
>
> [Linux Kernel](https://www.kernel.org/)
>
> [Bootlin: Elixir Cross Referencer](https://elixir.bootlin.com/linux/latest/source)
>
> [搭建高效的 Linux 开发环境](https://biscuitos.github.io/blog/Linux-debug-tools/)

## 赞赏一下吧 🙂

![MMU](/assets/PDB/BiscuitOS/kernel/HAB000036.jpg)
