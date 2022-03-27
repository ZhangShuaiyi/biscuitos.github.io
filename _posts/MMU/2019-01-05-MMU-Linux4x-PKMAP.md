---
layout: post
title:  "Linux 4.x 内核空间 PKMAP 虚拟内存地址"
date:   2019-01-05 15:56:30 +0800
categories: [MMU]
excerpt: Linux 4.x Kernel PKMAP Virtual Space.
tags:
  - MMU
---

> Architecture: i386 32bit Machine Ubuntu 16.04
>
> Linux version: 4.15.0-39-generic
>
> Email: BuddyZhang1 <buddy.zhang@aliyun.com>

# 目录

> - [PKMAP 虚拟内存区](#PKMAP 虚拟内存区)
>
> - [PKMAP 虚拟内存区中分配内存](#分配内存)
>
> - [PKMAP 虚拟内存实践](#PKMAP 虚拟内存实践)
>
> - [总结](#总结)
>
> - [附录](#附录)

--------------------------------------------

# <span id="PKMAP 虚拟内存区">PKMAP 虚拟内存区</span>

在 IA32 体系结构中，由于 CPU 的地址总线只有 32 位，在不开启 PAE 的情况下，CPU
可以访问 4G 的线性地址空间。Linux 采用了 3:1 的策略，即内核占用 1G 的线性地址
空间，用户占用 3G 的线性地址空间。如果 Linux 物理内存小于 1G 的空间，通常将线
性地址空间和物理空间一一映射，这样可以提供访问速度。但是，当 Linux 物理内存超
过 1G，内核的线性地址就不够用，所以，为了解决这个问题，Linux 把内核的虚拟地址
空间分作线性区和非线性区两个部分，线性区规定最大为 896M，剩下的 128M 为非线性
区。从而，线性区映射的物理内存都是低端内存，剩下的物理内存作为高端内存。

随着计算机技术的发展，计算机的实际物理内存越来越大，从而使得内核固定映射区 (线
性区，低端内存) 也越来越大。显然，如果不加以限制，当实际物理内存达到 1GB 时，
vmalloc 分配区 (非线性区) 将不复存在。于是以前开发的，调用 vmalloc() 的内核代
码也就不能再使用，显然为了兼容早期的内核代码，这是不允许的。

显然，出现上述文件的原因就是没有预料到实际物理内存可能超过 1GB，因此没有为内核
固定映射区的边界设定限制，而任由其随实际物理内存的增大而增大。解决上述问题的方
法就是：对内核固定映射区上限加以限制，使之不能随着物理内存的增加而任意增加。
Linux 规定，内核映射区的上边界最大不能超过 high_memory, high_memory 是一个小于
1G 的常数。当实际物理内存较大时，以 3G+highmem 为边界来确定虚拟内存区域。

> 例如
>     对于 IA32 系统，high_memory 的值就是 896M，于是 1GB 内核空间剩余下的 
>     128MB 为非线性映射区。这样就确保在任何情况下，内核都有足够的非线性映
>     射区以兼容早期代码并可以按普通方式访问实际物理内存的 1GB 以上的空间。

也就是说，高端内存的基本思想：借用一段虚拟地址空间，建立临时地址映射 (页表), 
使用完之后就释放，以此达到这段地址空间可以循环使用，访问所有的物理内存。高端
虚拟内存区域在 Linux 内核空间的分布图如下：

{% highlight ruby %}
4G +----------------+
   |                |
   +----------------+-- FIXADDR_TOP
   |                |
   |                | FIX_KMAP_END
   |     Fixmap     |
   |                | FIX_KMAP_BEGIN
   |                |
   +----------------+-- FIXADDR_START
   |                |
   |                |
   +----------------+--
   |                | A
   |                | |
   |   Persistent   | | LAST_PKMAP * PAGE_SIZE
   |    Mappings    | |
   |                | V
   +----------------+-- PKMAP_BASE
   |                |
   +----------------+-- VMALLOC_END / MODULE_END
   |                |
   |                |
   |    VMALLOC     |
   |                |
   |                |
   +----------------+-- VMALLOC_START / MODULE_VADDR
   |                | A
   |                | |
   |                | | VMALLOC_OFFSET
   |                | |
   |                | V
   +----------------+-- high_memory
   |                |
   |                |
   |                |
   | Mapping of all |
   |  physical page |
   |     frames     |
   |    (Normal)    |
   |                |
   |                |
   +----------------+-- MAX_DMA_ADDRESS
   |                |
   |      DMA       |
   |                |
   +----------------+
   |     .bss       |
   +----------------+
   |     .data      |
   +----------------+
   | 8k thread size |
   +----------------+
   |     .init      |
   +----------------+
   |     .text      |
   +----------------+ 0xC0008000
   | swapper_pg_dir |
   +----------------+ 0xC0004000
   |                |
3G +----------------+-- TASK_SIZE / PAGE_OFFSET
   |                |
   |                |
   |                |
0G +----------------+
{% endhighlight %}

习惯上，Linux 把内核虚拟空间 3G+high_memory ~ 4G-1 的这部分统称为高端虚拟内存。
根据应用的不同，高端内存区分 vmalloc 区，可持久映射区和临时映射区。

### PKMAP 可持久映射区

PKMAP 映射区也称为可持久映射区。当通过 alloc_page() 获得高端物理内存对于的 
page，内核专门为此保留一块线性地址空间，从 PKMAP_BASE 开始，用于映射高端物理
内存页，就是可持久映射区。在可持久映射区，可以通过调用函数 kmap() 在物理页框
与内核虚拟内存地址建立长期映射，这个空间通常为 4MB，最多能映射 1024 个页框，
最大页框数使用 LAST_PKMAP 宏表示。PKMAP 可映射的页框数稀少，所以为了加强页框
的使用率，应及时调用 kunmap() 函数将不使用的物理页框释放掉。

在 Linux 2.6 内核上，可持久映射虚拟地址范围是 4GB - 8M 到 4G-4M 之间。这个空
间和其他空间使用同样的页目录表，对于内核来说，就是 swapper_pg_dir，对于普通进
程来说，通过 CR3 寄存器指向。通常情况下，这个空间是 4M 大小，因此仅仅需要一个
页表就可以，内核通过 pkmap_page_table 寻址这个页表。

-----------------------------------------------------

# <span id="分配内存">PKMAP 虚拟内存区中分配内存</span>

IA32 体系结构中，Linux 4.x 可以使用 kmap() 函数从 PKMAP 可持久虚拟内存中获得
虚拟内存。函数使用方法如下：

{% highlight ruby %}
#if defined CONFIG_DEBUG_VA_KERNEL_KMAP && defined CONFIG_X86_32
    /*
     * PKMAP Virtual Space.
     * 0    3G                                                  4G
     * +----+----------+---------------------------------+------+
     * |    |          |                                 |      |
     * |    |          |   Persister Mappings (FIXMAP)   |      |
     * |    |          |                                 |      |
     * +----+----------+---------------------------------+------+
     *                 A                                 A
     *                 |                                 |
     *                 |                                 |
     *                 |                                 |
     *                 o                                 o
     *             PKMAP_BASE           PKMAP_BASE+(LAST_PKMAP * PAGE_SIZE))
     */

    struct page *high_page  = NULL;
    unsigned int *KMAP_init = NULL;

    /* Allocate a page from Highmem */
    high_page = alloc_pages(__GFP_HIGHMEM, 0);
    /* Map on Kmap */
    KMAP_int  = kmap(high_page);
    printk("[*]unsigned int *KMAP_int:       Address: %#08x\n",
                               (unsigned int)(unsigned long)KMAP_int);
    if (KMAP_int)
        kunmap(high_page);
    __free_pages(high_page, 0);
#endif
{% endhighlight %}

使用 alloc_pages() 函数和 __GFP_HIGHMEM 标志从高端物理内存分配一个物理页框，
然后使用 kmap 函数将这个物理页映射到 KMAP 区域，程序可以通过使用返回的地址，
匿名的方式访问这段内存。当不使用这部分虚拟内存，需要使用 kunmap() 函数进行释放。

--------------------------------------

# <span id="PKMAP 虚拟内存实践">VMALLOC 虚拟内存实践</span>

BiscuitOS 提供了相关的实例代码，开发者可以使用如下命令：
首先，开发者先准备 BiscuitOS 系统，内核版本 linux 1.0.1.2。开发可以参照文档构
建 BiscuitOS 调试环境：

[Linux 1.0.1.2 内核构建方法](/blog/Linux1.0.1.2_ext2fs_Usermanual/)

由于代码需要运行在 IA32 系统上，如果开发者的系统不是 IA32，那么可以按照下面的
教程搭建一个 IA32 虚拟机：

[Ubuntu IA32 虚拟环境build](/blog/Linux1.0.1.2_ext2fs_Usermanual/)

接着，开发者配置内核，使用如下命令：

{% highlight ruby %}
cd BiscuitOS
make clean
make update
make linux_1_0_1_2_ext2_defconfig
make
cd BiscuitOS/kernel/linux_1.0.1.2/
make clean
make menuconfig
{% endhighlight %}

由于 BiscuitOS 的内核使用 Kbuild 构建起来的，在执行完 make menuconfig 之后，系
统会弹出内核配置的界面，开发者根据如下步骤进行配置：

![MMU](/assets/PDB/BiscuitOS/kernel/MMU000003.png)

选择 kernel hacking，回车

![MMU](/assets/PDB/BiscuitOS/kernel/MMU000004.png)

选择 Demo Code for variable subsystem mechanism, 回车

![MMU](/assets/PDB/BiscuitOS/kernel/MMU000005.png)

选择 MMU(Memory Manager Unit) on X86 Architecture, 回车

![MMU](/assets/PDB/BiscuitOS/kernel/MMU000433.png)

选择 Debug MMU(Memory Manager Unit) mechanism on X86 Architecture 之后选择 
Addressing Mechanism  回车

![MMU](/assets/PDB/BiscuitOS/kernel/MMU000434.png)

选择 Virtual address, 回车

![MMU](/assets/PDB/BiscuitOS/kernel/MMU000525.png)

选择 Virtual address and Virtual space 之后，接着选择 Choice Kernel/User 
Virtual Address Space, 回车。

![MMU](/assets/PDB/BiscuitOS/kernel/MMU000526.png)

该选项用于选择程序运行在用户空间还是内核空间，这里选择内核空间。选择 Kernel 
Virtual Address Space. 回车之后按 Esc 退出。

![MMU](/assets/PDB/BiscuitOS/kernel/MMU000536.png)

最后开发者选择 .data segment,下拉菜单打开后，选择 KMAP Virtual Space 
(kmap) 选项，回车保存并退出。

运行实例代码，使用如下代码：

{% highlight ruby %}
cd BiscuitOS/kernel/linux_1.0.1.2/
make 
cd tools/demo/mmu/addressing/virtual_address/data/kernel/
{% endhighlight %}

如果开发者的主机本身即是 IA32，那么使用如下命令安装并运行模块程序

{% highlight ruby %}
sudo insmod kern_data.ko
dmesg | tail -n 20
{% endhighlight %}

如果开发者的主机不是 IA32，那么使用如下命令运行并安装模块

{% highlight ruby %}
cp kern_data.c /虚拟机指定目录
cp .tmp/Makefile /虚拟机指定目录
{% endhighlight %}

开发者可以通过多种方式将上面两个文件拷贝到虚拟机指定目录下，然后在虚拟机上执行
如下命令

{% highlight ruby %}
make
make install
{% endhighlight %}

源码如下：

![MMU](/assets/PDB/BiscuitOS/kernel/MMU000537.png)

Makefile

{% highlight ruby %}
# Module on higher linux version
obj-m += kern_data.o

kern_dataDIR ?= /lib/modules/$(shell uname -r)/build

PWD       := $(shell pwd)

ROOT := $(dir $(M))
DEMOINCLUDE := -I$(ROOT)../include -I$(ROOT)/include

all:
        $(MAKE) -C $(kern_dataDIR) M=$(PWD) modules

install:
        @sudo insmod kern_data.ko
        @dmesg | tail -n 20
        @sudo rmmod kern_data

clean:
        @rm -rf *.o *.o.d *~ core .depend .*.cmd *.ko *.ko.unsigned *.mod.c .tmp_ \
    .cache.mk *.save *.bak Modules.* modules.order Module.* *.b

CFLAGS_kern_data.o := -Wall $(DEMOINCLUDE)
CFLAGS_kern_data.o += -DCONFIG_DEBUG_VA_KERNEL_KMAP -fno-common
{% endhighlight %}

运行结果如图：

![MMU](/assets/PDB/BiscuitOS/kernel/MMU000538.png)

从上面数据可知，使用 kmap() 函数分配的虚拟内存位于 0xFFC00000 到 0xFFE00000 之
间，所以分配成功。

---------------------------------------------

# <span id="总结">总结</span>

KMAP 提供了可持久的映射方法，所谓可持久映射指其虚拟地址到物理地址映射的页表，
可以被多个进程可见。由于 KMAP 只提供了 4MB 的映射空间，内核应该在不使用时，及
时使用 kunmap() 函数进行释放。

-----------------------------------------------

# <span id="附录">附录</span>

[Linux 的内核空间（低端内存，高端内存）](https://blog.csdn.net/qq_38410730/article/details/81105132)

赞赏一下吧 🙂

![MMU](/assets/PDB/BiscuitOS/kernel/HAB000036.jpg)
