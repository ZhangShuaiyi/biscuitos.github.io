---
layout: post
title:  "__bitmap_andnot"
date:   2019-06-10 05:30:30 +0800
categories: [HW]
excerpt: BITMAP __bitmap_andnot().
tags:
  - Tree
---

![DTS](/assets/PDB/BiscuitOS/kernel/IND00000B.jpg)

> [Github: __bitmap_andnot](https://github.com/BiscuitOS/HardStack/tree/master/Algorithem/bitmap/API/__bitmap_andnot)
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
int __bitmap_andnot(unsigned long *dst, const unsigned long *bitmap1,
                                const unsigned long *bitmap2, unsigned int bits)
{
        unsigned int k;
        unsigned int lim = bits/BITS_PER_LONG;
        unsigned long result = 0;

        for (k = 0; k < lim; k++)
                result |= (dst[k] = bitmap1[k] & ~bitmap2[k]);
        if (bits % BITS_PER_LONG)
                result |= (dst[k] = bitmap1[k] & ~bitmap2[k] &
                           BITMAP_LAST_WORD_MASK(bits));
        return result != 0;
}
EXPORT_SYMBOL(__bitmap_andnot);
{% endhighlight %}

__bitmap_andnot() 函数用于两个 bitmap 进行与非运算。参数 dst 用于存储与非的
结果；参数 bitmap1 指向第一个 bitmap；参数 bitmap2 指向取反的 bitmap；参数
bits 代表需要进行与非的位数。函数首先将 bits 除以 BITS_PER_LONG 计算 bits
使用了几个 long 变量，然后使用 for 循环从两个 bitmap 的 LSB long 开始，
将第一个 bitmap 与第二个 bitmap 的取反进行相与，结果存储在 dst 对应的位置。
如果 bits 的数量没有按 long 对齐，那么将多余的 bit 对应的 bitmap 进行
与非运算，并使用 BITMAP_LAST_WORD_MASK() 函数计算对应的掩码，以此获得
多出位的与非运算。最后判断 result 的值是否为 0 并返回判断结果。

> - [BITMAP_LAST_WORD_MASK](/blog/BITMAP_BITMAP_LAST_WORD_MASK/)

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
	unsigned long dst;
	unsigned long bitmap1 = 0xffff0000;
	unsigned long bitmap2 = 0x000fffff;

	__bitmap_andnot(&dst, &bitmap1, &bitmap2, 32);
	printk("DST: %#lx\n", dst);

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
+       bool "__bitmap_andnot"
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
            [*]__bitmap_andnot()
{% endhighlight %}

具体过程请参考：

> [Linux 5.0 开发环境搭建 -- 驱动配置](/blog/Linux-5.0-arm32-Usermanual/#%E9%A9%B1%E5%8A%A8%E9%85%8D%E7%BD%AE)

#### <span id="驱动编译">驱动编译</span>

驱动编译也请参考下面文章关于驱动编译一节：

> [Linux 5.0 开发环境搭建 -- 驱动编译](/blog/Linux-5.0-arm32-Usermanual/#%E7%BC%96%E8%AF%91%E9%A9%B1%E5%8A%A8)

#### <span id="驱动运行">驱动运行</span>

驱动的运行，请参考下面文章中关于驱动运行一节：

> [Linux 5.0 开发环境搭建 -- 驱动运行](/blog/Linux-5.0-arm32-Usermanual/#%E9%A9%B1%E5%8A%A8%E8%BF%90%E8%A1%8C)

启动内核，并打印如下信息：

{% highlight bash %}
usbcore: registered new interface driver usbhid
usbhid: USB HID core driver
DST: 0xfff00000
aaci-pl041 10004000.aaci: ARM AC'97 Interface PL041 rev0 at 0x10004000, irq 24
aaci-pl041 10004000.aaci: FIFO 512 entries
oprofile: using arm/armv7-ca9
{% endhighlight %}

#### <span id="驱动分析">驱动分析</span>

bitmap 的与非操作。

-----------------------------------------------

# <span id="附录">附录</span>

> [BiscuitOS Home](https://biscuitos.github.io/)
>
> [BiscuitOS Driver](/blog/BiscuitOS_Catalogue/)
>
> [BiscuitOS Kernel Build](/blog/Kernel_Build/)
>
> [Linux Kernel](https://www.kernel.org/)
>
> [Bootlin: Elixir Cross Referencer](https://elixir.bootlin.com/linux/latest/source)
>
> [搭建高效的 Linux 开发环境](/blog/Linux-debug-tools/)

## 赞赏一下吧 🙂

![MMU](/assets/PDB/BiscuitOS/kernel/HAB000036.jpg)
