---
layout: post
title:  "Linux 4.x 内核空间 FIXUP 固定映射和临时映射虚拟内存"
date:   2019-01-05 16:10:30 +0800
categories: [MMU]
excerpt: Linux 4.x Kernel FIXUP Virtual Space.
tags:
  - MMU
---

> Architecture: i386 32bit Machine Ubuntu 16.04
>
> Linux version: 4.15.0-39-generic
>
> Email: BuddyZhang1 <buddy.zhang@aliyun.com>

# 目录

> - [FIXUP 虚拟内存区](#FIXUP 虚拟内存区)
>
> - [FIXUP 虚拟内存区中分配内存](#分配内存)
>
> - [FIXUP 虚拟内存实践](#FIXUP 虚拟内存实践)
>
> - [总结](#总结)
>
> - [附录](#附录)

--------------------------------------------

# <span id="FIXUP 虚拟内存区">FIXUP 虚拟内存区</span>

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

### FIXUP 固定和临时映射区

内核在 FIXADDR_START 到 FIXUP_TOP 之间保留了一些虚拟地址空间用于特殊需求。这个
虚拟空间称为“固定映射空间”或 “FIXUP 固定映射”。内核在编译阶段就设置好该区域的
虚拟内存布局，所以称为固定映射。内核也在该区域为每个 CPU 核留了一段可以零时映
射的区域，该区域称为临时映射虚拟内存区，该区域起始索引是 FIX_KMAP_BEGIN，终止
索引是 FIX_KMAP_END，内核可以通过 __fix_to_virt() 和 set_fixmap() 函数进行该虚
拟内存区域的分配。固定映射是与物理地址空间中的固定页关联的虚拟地址空间项，但具
体关联的页帧可以自由选择。它与通过固定公式与物理内存关联的直接映射页相反，虚拟
固定映射地址与物理内存位置之间的关联可以自行定义，关联建立后内核总是会看到。

下表是 X86 的 FIXUP 固定映射分配源码：

{% highlight ruby %}
/*
* Here we define all the compile-time 'special' virtual
* addresses. The point is to have a constant address at
* compile time, but to set the physical address only
* in the boot process.
* for x86_32: We allocate these special addresses
* from the end of virtual memory (0xfffff000) backwards.
* Also this lets us do fail-safe vmalloc(), we
* can guarantee that these special addresses and
* vmalloc()-ed addresses never overlap.
*
* These 'compile-time allocated' memory buffers are
* fixed-size 4k pages (or larger if used with an increment
* higher than 1). Use set_fixmap(idx,phys) to associate
* physical memory with fixmap indices.
*
* TLB entries of such buffers will not be flushed across
* task switches.
*/
enum fixed_addresses {
#ifdef CONFIG_X86_32
    FIX_HOLE,
#else
#ifdef CONFIG_X86_VSYSCALL_EMULATION
    VSYSCALL_PAGE = (FIXADDR_TOP - VSYSCALL_ADDR) >> PAGE_SHIFT,
#endif
#endif
    FIX_DBGP_BASE,
    FIX_EARLYCON_MEM_BASE,
#ifdef CONFIG_PROVIDE_OHCI1394_DMA_INIT
    FIX_OHCI1394_BASE,
#endif
#ifdef CONFIG_X86_LOCAL_APIC
    FIX_APIC_BASE,    /* local (CPU) APIC) -- required for SMP or not */
#endif
#ifdef CONFIG_X86_IO_APIC
    FIX_IO_APIC_BASE_0,
    FIX_IO_APIC_BASE_END = FIX_IO_APIC_BASE_0 + MAX_IO_APICS - 1,
#endif
#ifdef CONFIG_X86_32
    FIX_KMAP_BEGIN,    /* reserved pte's for temporary kernel mappings */
    FIX_KMAP_END = FIX_KMAP_BEGIN+(KM_TYPE_NR*NR_CPUS)-1,
#ifdef CONFIG_PCI_MMCONFIG
    FIX_PCIE_MCFG,
#endif
#endif
#ifdef CONFIG_PARAVIRT
    FIX_PARAVIRT_BOOTMAP,
#endif
    FIX_TEXT_POKE1,    /* reserve 2 pages for text_poke() */
    FIX_TEXT_POKE0, /* first page is last, because allocation is backward */
#ifdef    CONFIG_X86_INTEL_MID
    FIX_LNW_VRTC,
#endif

#ifdef CONFIG_ACPI_APEI_GHES
    /* Used for GHES mapping from assorted contexts */
    FIX_APEI_GHES_IRQ,
    FIX_APEI_GHES_NMI,
#endif

    __end_of_permanent_fixed_addresses,

    /*
     * 512 temporary boot-time mappings, used by early_ioremap(),
     * before ioremap() is functional.
     *
     * If necessary we round it up to the next 512 pages boundary so
     * that we can have a single pgd entry and a single pte table:
     */
#define NR_FIX_BTMAPS        64
#define FIX_BTMAPS_SLOTS    8
#define TOTAL_FIX_BTMAPS    (NR_FIX_BTMAPS * FIX_BTMAPS_SLOTS)
    FIX_BTMAP_END =
     (__end_of_permanent_fixed_addresses ^
      (__end_of_permanent_fixed_addresses + TOTAL_FIX_BTMAPS - 1)) &
     -PTRS_PER_PTE
     ? __end_of_permanent_fixed_addresses + TOTAL_FIX_BTMAPS -
       (__end_of_permanent_fixed_addresses & (TOTAL_FIX_BTMAPS - 1))
     : __end_of_permanent_fixed_addresses,
    FIX_BTMAP_BEGIN = FIX_BTMAP_END + TOTAL_FIX_BTMAPS - 1,
#ifdef CONFIG_X86_32
    FIX_WP_TEST,
#endif
#ifdef CONFIG_INTEL_TXT
    FIX_TBOOT_BASE,
#endif
    __end_of_fixed_addresses
};
{% endhighlight %}

-----------------------------------------------------

# <span id="分配内存">FIXUP 虚拟内存区中分配内存</span>

IA32 体系结构中，Linux 4.x 可以使用 fix_to_virt() 函数从 FIXUP 的 
FIX_KMAP_BEGIN 到 FIX_KMAP_END 之间的临时映射中获得虚拟内存。函数使用方法如下：

{% highlight ruby %}
#if defined CONFIG_DEBUG_VA_KERNEL_FIXMAP && defined CONFIG_X86_32
    /*
     * FIXMAP Virtual Space.
     * 0    3G                                                  4G
     * +----+---------------------------+----------------+------+
     * |    |                           |                |      |
     * |    |                           |  Fixmap Space  |      |
     * |    |                           |                |      |
     * +----+---------------------------+----------------+------+
     *                                  A                A
     *                                  |                |
     *                                  |                |
     *                                  |                |
     *                                  o                o
     *                             FIXADDR_START    FIXADDR_TOP
     *
     * Here define all the compile-time 'special' virtual addresses. The
     * point is to have a constant address at compile time, but to set the
     * physical address only in the boot process.
     *
     * For IA32: We allocate these speical addresses from the end of virtual
     * memory (0xfffff000) backwards. Also this lets us do fail-safe
     * vmalloc(), we can gurarntee that these special addresses and
     * vmalloc()-ed addresses never overlap.
     *
     * These 'compile-time allocated' memory buffers are fixed-size 4K pages
     * (or larger if used with an increament higher than 1). Use set
     * fixmap(idx, phys) to associate physical memory with fixmap indices.
     *
     * TLB entries of such buffers will not be flushed accross task switch.
     */

    struct page *high_page;
    int idx, type;
    unsigned long vaddr;

    /* Allocate a physical page */
    high_page = alloc_page(__GFP_HIGHMEM);

    /* Obtain current CPU's FIX_KMAP_BEGIN */
    type = kmap_atomic_idx_push();
    idx  = FIX_KMAP_BEGIN + type + KM_TYPE_NR * smp_processor_id();

    /* Obtain fixmap virtual address by index */
    vaddr = fix_to_virt(idx);
    /* Associate fixmap virtual address with physical address */
    set_fixmap(idx, page_to_phys(high_page));

    printk("[*]unsignd long vaddr:       Address: %#08x\n",
                               (unsigned int)(unsigned long)vaddr);

    /* Remove associate with fixmap */
    clear_fixmap(idx);
#endif
{% endhighlight %}

通过 alloc_page() 函数和 __GFP_HIGHMEM 标志从内核中获得一个高端物理内存页框，
然后通过 kmap_atomic_idx_push() 函数，从 FIX_KMAP_BEGIN 到 FIX_KMAP_END 中获
得当前 CPU 的临时映射区所以。最后通过 __fix_to_virt() 和 set_fixmap() 函数将
这个高端页映射到临时映射区内，内核就可以使用这块虚拟内存区。如果不在使用这部
分内存区，使用 clear_fixmap() 和 free_page() 是否这段虚拟内存和物理页。

--------------------------------------

# <span id="FIXUP 虚拟内存实践">FIXUP 虚拟内存实践</span>

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

![MMU](/assets/PDB/BiscuitOS/kernel/MMU000539.png)

最后开发者选择 .data segment,下拉菜单打开后，选择 Fixmap Virtual Space 
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

![MMU](/assets/PDB/BiscuitOS/kernel/MMU000540.png)

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
CFLAGS_kern_data.o += -DCONFIG_DEBUG_VA_KERNEL_FIXMAP -fno-common
{% endhighlight %}

运行结果如图：

![MMU](/assets/PDB/BiscuitOS/kernel/MMU000541.png)

从上面数据可知，使用 __fix_to_virt 函数分配的虚拟内存位于 0xFFF15000 到 
0xFFFFF000 之间，所以从临时映射虚拟内存中分配成功。

---------------------------------------------

# <span id="总结">总结</span>

高端内存中，FIXUP 作为编译阶段有内核决定的固定映射虚拟内存区，其从 
FIXADDR_START 延伸到 FIXADDR_TOP，保留给内核特定功能使用。在这些固定映射中，
存在着一块临时映射虚拟区，这块区域从 FIX_KMAP_BEGIN 到 FIX_KMAP_END，内核定义
这块区域之后，可以供系统特定任务需求。

-----------------------------------------------

# <span id="附录">附录</span>

[Linux 的内核空间（低端内存，高端内存）](https://blog.csdn.net/qq_38410730/article/details/81105132)

赞赏一下吧 🙂

![MMU](/assets/PDB/BiscuitOS/kernel/HAB000036.jpg)
