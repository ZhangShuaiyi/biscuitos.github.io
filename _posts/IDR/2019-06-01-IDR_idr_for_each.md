---
layout: post
title:  "idr_for_each"
date:   2019-06-01 05:30:30 +0800
categories: [HW]
excerpt: IDR idr_for_each().
tags:
  - Tree
---

![DTS](/assets/PDB/BiscuitOS/kernel/IND00000T.jpg)

> [Github: idr_for_each](https://github.com/BiscuitOS/HardStack/tree/master/Algorithem/IDR/API/idr_for_each)
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
int idr_for_each(const struct idr *idr,
                int (*fn)(int id, void *p, void *data), void *data)
{
        struct radix_tree_iter iter;
        void __rcu **slot;
        int base = idr->idr_base;

        radix_tree_for_each_slot(slot, &idr->idr_rt, &iter, 0) {
                int ret;
                unsigned long id = iter.index + base;

                if (WARN_ON_ONCE(id > INT_MAX))
                        break;
                ret = fn(id, rcu_dereference_raw(*slot), data);
                if (ret)
                        return ret;
        }

        return 0;
}
EXPORT_SYMBOL(idr_for_each);
{% endhighlight %}

idr_for_each() 函数用于遍历 IDR 所有 ID 对应的指针，也就是遍历 radix-tree 已经
使用 slots。参数 idr 指向 IDR 根节点；参数 fn 是一个回调函数; data 参数为传递给
fn 参数的私有数据。函数首先调用 radix_tree_for_each_slot() 函数遍历 radix-tree
所有已经在使用的 slot，每次遍历到 slot 的时候，调用 fn 函数运行。遍历完之后返回 0.

> - [radix_tree_for_each_slot](/blog/RADIX-TREE_radix_tree_for_each_slot/)

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
 * IDR.
 *
 * (C) 2019.06.04 <buddy.zhang@aliyun.com>
 *
 * This program is free software; you can redistribute it and/or modify
 * it under the terms of the GNU General Public License version 2 as
 * published by the Free Software Foundation.
 */

#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/mm.h>

/* header of radix-tree */
#include <linux/idr.h>

/* private node */
struct node {
	const char *name;
};

/* Root of IDR */
static DEFINE_IDR(BiscuitOS_idr);

/* node set */
static struct node node0 = { .name = "IDA" };
static struct node node1 = { .name = "IDB" };
static struct node node2 = { .name = "IDC" };
static struct node node3 = { .name = "IDD" };
static struct node node4 = { .name = "IDE" };

/* ID array */
#define IDR_ARRAY_SIZE	5
static int idr_array[IDR_ARRAY_SIZE];

static int count_idrs(int id, void *p, void *data)
{
	int *countp = data;

	*countp = *countp + 1;
	return 0;
}

static __init int idr_demo_init(void)
{
	struct node *np;
	int count = 1;
	int id;

	/* proload for idr_alloc */
	idr_preload(GFP_KERNEL);

	/* Allocate a id from IDR */
	idr_array[0] = idr_alloc_cyclic(&BiscuitOS_idr, &node0, 1,
							INT_MAX, GFP_ATOMIC);
	idr_array[1] = idr_alloc_cyclic(&BiscuitOS_idr, &node1, 1,
							INT_MAX, GFP_ATOMIC);
	idr_array[2] = idr_alloc_cyclic(&BiscuitOS_idr, &node2, 1,
							INT_MAX, GFP_ATOMIC);
	idr_array[3] = idr_alloc_cyclic(&BiscuitOS_idr, &node3, 1,
							INT_MAX, GFP_ATOMIC);
	idr_array[4] = idr_alloc_cyclic(&BiscuitOS_idr, &node4, 1,
							INT_MAX, GFP_ATOMIC);

	/* Interate over all slots */
	idr_for_each_entry(&BiscuitOS_idr, np, id)
		printk("%s's ID %d\n", np->name, id);

	/* Iterate through all stored pointers. */
	idr_for_each(&BiscuitOS_idr, count_idrs, &count);
	printk("Count IDs: %d\n", count);

	/* end preload section started with idr_preload() */
	idr_preload_end();

	return 0;
}
device_initcall(idr_demo_init);
{% endhighlight %}

#### <span id="驱动安装">驱动安装</span>

驱动的安装很简单，首先将驱动放到 drivers/BiscuitOS/ 目录下，命名为 atomic.c，
然后修改 Kconfig 文件，添加内容参考如下：

{% highlight bash %}
diff --git a/drivers/BiscuitOS/Kconfig b/drivers/BiscuitOS/Kconfig
index 4edc5a5..1a9abee 100644
--- a/drivers/BiscuitOS/Kconfig
+++ b/drivers/BiscuitOS/Kconfig
@@ -6,4 +6,14 @@ if BISCUITOS_DRV
config BISCUITOS_MISC
        bool "BiscuitOS misc driver"
+config BISCUITOS_IDR
+       bool "IDR"
+
+if BISCUITOS_IDR
+
+config DEBUG_BISCUITOS_IDR
+       bool "idr_for_each"
+
+endif # BISCUITOS_IDR
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
+obj-$(CONFIG_BISCUITOS_IDR)     += idr.o
--
{% endhighlight %}

#### <span id="驱动配置">驱动配置</span>

驱动配置请参考下面文章中关于驱动配置一节。在配置中，勾选如下选项，如下：

{% highlight bash %}
Device Driver--->
    [*]BiscuitOS Driver--->
        [*]IDR
            [*]idr_for_each()
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
IDA's ID 1
IDB's ID 2
IDC's ID 3
IDD's ID 4
IDE's ID 5
Count IDs: 6
aaci-pl041 10004000.aaci: ARM AC'97 Interface PL041 rev0 at 0x10004000, irq 24
aaci-pl041 10004000.aaci: FIFO 512 entries
oprofile: using arm/armv7-ca9
{% endhighlight %}

#### <span id="驱动分析">驱动分析</span>

遍历 IDR 中所有与 ID 绑定的指针。

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
