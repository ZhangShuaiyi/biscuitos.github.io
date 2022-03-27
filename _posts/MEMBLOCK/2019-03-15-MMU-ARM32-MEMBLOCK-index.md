---
layout:             post
title:              "MEMBLOCK 内存分配器"
date:               2019-03-15 15:15:30 +0800
categories:         [MMU]
excerpt:            MEMBLOCK 内存分配器.
tags:
  - MMU
---

![MMU](https://gitee.com/BiscuitOS/GIFBaseX/raw/master/RPI/GIF000202.gif)

![](/assets/PDB/RPI/RPI100100.png)

#### 目录

> - [MEMBLOCK 内存分配器原理](#MEMBLOCK 原理)
>
> - [MEMBLOCK 内存分配器最小实践](#MEMBLOCK 内存分配器最小实践)
>
> - [MEMBLOCK 内存分配器的使用](#MEMBLOCK 内存分配器的使用)
>
> - [MEMBLOCK 内存分配器源码分析](#MEMBLOCK 源码分析)
>
> - [MEMBLOCK 内存分配器调试](#MEMBLOCK 调试)
>
> - [MEMBLOCK 分配器时间轴](#A)
>
> - [MEMBLOCK 分配器历史补丁](#B)
>
> - [MEMBLOCK 内存分配器进阶研究](#内存分配器进阶实践)
>
> - [MEMBLOCK API List](#MEMBLOCK_API-LIST)
>
> - [附录](#附录)

--------------------------------------------------------------
<span id="MEMBLOCK 原理"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000T.jpg)

#### MEMBLOCK 内存分配器原理

MEMBLOCK 内存分配器作为 arm32 早期的内存管理器，用于维护系统可用的物理内存。
系统启动过程中，可以使用 MEMBLOCK 内存分配器获得所需的物理内存，也可以将特定
物理内存预留给特殊功能使用。MEMBLOCK 内存分配器逻辑框架简单易用，框架中将将物理
内存和已分配的物理内存维护在不同的数据结构中，以此实现 MEMBLOCK 分配器对物理
内存的分配和回收等操作。MEMBLOCK 分配器基本逻辑框架如下图：

{% highlight bash %}
MEMBLOCK


                                         struct memblock_region
                       struct            +------+------+--------+------+
                       memblock_type     |      |      |        |      |
                       +----------+      | Reg0 | Reg1 | ...    | Regn |
                       |          |      |      |      |        |      |
                       | regions -|----->+------+------+--------+------+
                       | cnt      |      [memblock_memory_init_regions]
                       |          |
 struct           o--->+----------+
 memblock         |
 +-----------+    |
 |           |    |
 | memory   -|----o
 | reserved -|----o
 |           |    |                      struct memblock_region
 +-----------+    |    struct            +------+------+--------+------+
                  |    memblock_type     |      |      |        |      |
                  o--->+----------+      | Reg0 | Reg1 | ...    | Regn |
                       |          |      |      |      |        |      |
                       | regions -|----->+------+------+--------+------+
                       | cnt      |      [memblock_reserved_init_regions]
                       |          |
                       +----------+
{% endhighlight %}

从上面的逻辑图可以知道，MEMBLOCK 分配器使用一个 struct memblock 结构维护着两种内存，
其中成员 memory 维护着可用物理内存区域；成员 reserved 维护着操作系统预留的内存区域。
每个区域使用数据结构 struct memblock_type 进行管理，其成员 regions 负责维护该类型内
存的所有内存区，每个内存区使用数据结构 struct memblock_region 进行维护。

基于这种逻辑框架，MEMBLOCK 内存分配器为内核提供了不同的接口，以便其他模块在内核
启动阶段能够便捷分配到所需的物理内存，这些接口可以分为以下几类：

> - 物理内存的分配
>
> - 物理内存的释放回收
>
> - 物理内存信息的动态统计和查看

更多 MEMBLOCK 内存分配器原理，请看源码分析部分。

--------------------------------------------------------------

<span id="MEMBLOCK 内存分配器最小实践"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000Q.jpg)

#### MEMBLOCK 内存分配器最小实践

为了让开发者对 MEMBLOCK 内存分配器有更多的认识，开发者可以选择下面任何一个
实践主题进行实践，推荐多实践:

> - [MEMBLOCK 分配器 -- 从 MEMBLOCK 分配器中获得一块可用物理内存](/blog/MMU-ARM32-MEMBLOCK-memblock_alloc_range/)
>
> - [MEMBLOCK 分配器 -- 从 MEMBLOCK 分配器中释放一块物理内存](/blog/MMU-ARM32-MEMBLOCK-memblock_free/)
>
> - [MEMBLOCK 分配器 -- 将一块物理内存预留](/blog/MMU-ARM32-MEMBLOCK-memblock_reserve/)

---------------------------------------------------------------
<span id="MEMBLOCK 内存分配器的使用"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000R.jpg)

#### MEMBLOCK 内存分配器的使用

MEMBLOCK 分配器提供了很多接口供其他模块使用，开发者可以参考本节内容来
了解 MEMBLOCK 分配器的使用方法。

> - [分配物理内存](#分配物理内存)
>
> - [释放物理内存](#释放物理内存)
>
> - [添加可用物理内存](#添加可用物理内存)
>
> - [添加预留区](#添加预留区)
>
> - [遍历内存区](#遍历内存区)
>
> - [MEMBLOCK 分配器信息](#MEMBLOCK 分配器信息)

###### <span id="分配物理内存">分配物理内存</span>

MEMBLOCK 分配器所管理的是系统可用的物理内存，在系统启动初期，即
start_kernel->setup_arch->setup_machine_fdt() 函数之后就可以使用 MEMBLOCK
分配器分配所需的内存，其提供的 API 以及 API 的具体实践如下：

> - [memblock_alloc_base: 从某段物理地址之前分配特定长度的物理内存](/blog/MMU-ARM32-MEMBLOCK-memblock_alloc_base/)
>
> - [memblock_alloc_range: 从指定范围内分配特定长度的物理内存](/blog/MMU-ARM32-MEMBLOCK-memblock_alloc_range/)
>
> - [__memblock_alloc_base: 从指定地址之前开始分配物理内存](/blog/MMU-ARM32-MEMBLOCK-__memblock_alloc_base/)

###### <span id="释放物理内存">释放物理内存</span>

当使用完从 MEMBLOCK 分配器中申请的内存时，可以调用函数将这些物理内存归还给
MEMBLOCK 分配器，分配器就将这些物理内存从预留区中移除，然后放入到可用物理
内存区供其他模块使用，其提供的 API 以及 API 的具体实践如下：

> - [memblock_free: 释放一段物理内存](/blog/MMU-ARM32-MEMBLOCK-memblock_free/)
>
> - [memblock_remove: 从可用物理内存区内移除一段物理内存](/blog/MMU-ARM32-MEMBLOCK-memblock_remove/)

###### <span id="添加可用物理内存">添加可用物理内存</span>

MEMBLOCK 分配器初始化阶段或正常使用过程中需要往系统添加新的可用物理内存，
即往 MEMBLOCK 分配器的可用物理内存区添加新的物理块，其提供的 API 以及 API 的
具体实践如下：

> - [memblock_add: 往 MEMBLOCK 的可用内存区添加一块物理内存区块](/blog/MMU-ARM32-MEMBLOCK-memblock_add/)
>
> - [memblock_add_range: 往 MEMBLOCK 的可用内存区添加一块物理内存区块](/blog/MMU-ARM32-MEMBLOCK-memblock_add_range/)

###### <span id="添加预留区">添加预留区</span>

内核启动过程中，需要将某些物理内存预留给特定功能使用，这时可以使用 MEMBLOCK
分配器将这些物理内存区块加入到预留区维护起来，其提供的 API 以及 API 的具体实践如下：

> - [memblock_reserve: 将一块物理块加入预留区](/blog/MMU-ARM32-MEMBLOCK-memblock_reserve/)
>
> - [memblock_add_range: 往 MEMBLOCK 的预留区添加一块物理内存区块](/blog/MMU-ARM32-MEMBLOCK-memblock_add_range/)

###### <span id="遍历内存区">遍历内存区</span>

在内核中，有的时候需要遍历某个内存区内的所有内存区块，以此对各内存区进行安全合理
的操作，MEMBLOCK 分配器也提供了很多 API 以及 API 实践，如下：

> - [for_each_memblock: 遍历内存区内的所有内存区块](/blog/MMU-ARM32-MEMBLOCK-for_each_memblock/)
>
> - [for_each_free_mem_range: 正序遍历所有可用物理内存区块](/blog/MMU-ARM32-MEMBLOCK-for_each_free_mem_range/)
>
> - [for_each_free_mem_range_reverse: 倒序遍历所有可用物理内存区块](/blog/MMU-ARM32-MEMBLOCK-for_each_free_mem_range_reverse/)
>
> - [for_each_mem_range: 正序遍历所有可用物理内存区块](/blog/MMU-ARM32-MEMBLOCK-for_each_mem_range/)
>
> - [for_each_reserved_mem_region: 遍历预留区内的所有内存区块](/blog/MMU-ARM32-MEMBLOCK-for_each_reserved_mem_region/)

###### <span id="MEMBLOCK 分配器信息">MEMBLOCK 分配器信息</span>

有时需要通过 MEMBLOCK 分配器获得关于物理内存等消息，MEMBLOCK 分配器也提供
了很多 API 供使用，如下：

> - [memblock_phys_mem_size: 获得可用物理内存的总体积](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> - [memblock_reserved_size: 获得预留区内存的总体积](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> - [memblock_start_of_DRAM: 获得 DRAM 的起始物理地址](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> - [memblock_end_of_DRAM: 获得 DRAM 的结束物理地址](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> - [memblock_is_reserved: 检查某个物理地址是否位于预留区内](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> - [memblock_is_memory: 检查某个物理地址是否位于可用内存区](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> - [memblock_is_region_memory: 检查某段内存区是否属于可用内存区](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> - [memblock_is_region_reserved: 检查某段内存区块是否属于预留区](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> - [memblock_get_current_limit: 获得 MEMBLOCK 的 limit](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> - [memblock_set_current_limit: 设置 MEMBLOCK 的 limit](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> - [memblock_is_hotpluggable: 检查内存区块是否支持热插拔](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> - [memblock_is_mirror: 检查内存区块是否支持 mirror](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> - [memblock_is_nomap: 检查内存区块是否支持 nomap](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> - [memblock_get_region_node: 获得内存区块的 NUMA 号](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> - [memblock_set_region_node: 设置内存区块的 NUMA 号](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> - [memblock_bottom_up: 获得 MEMBLOCK 分配的方向](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> - [memblock_set_bottom_up: 设置 MEMBLOCK 分配的方向](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)

--------------------------------------------------------------
<span id="MEMBLOCK 源码分析"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000H.jpg)

#### MEMBLOCK 源码分析

> - [MEMBLOCK 内存分配器构建](#MEMBLOCK 内存分配器构建)
>
>   - [第一层数据结构](#第一层数据结构)
>
>   - [第二层数据结构](#第二层数据结构)
>
>   - [第三层数据结构](#第三层数据结构)
>
> - [MEMBLOCK 内存分配器使用](#MEMBLOCK 内存分配器使用)
>
>   - [构建可用物理内存](#构建可用物理内存)
>
>   - [adjust_lowmem_bounds](#adjust_lowmem_bounds)
>
>   - [arm_memblock_init](#arm_memblock_init)
>
>   - [bootmem_init](#bootmem_init)

###### <span id="MEMBLOCK 内存分配器构建">MEMBLOCK 内存分配器构建</span>

MEMBLOCK 内存分配器的创建为 /mm/memblock.c 文件中，在内核镜像加载到内存
之后，CPU 的执行权交给 Linux 之后，Linux 就根据这个文件，构建最原始的
MEMBLOCK 内存分配器，接下来分析 MEMBLOCK 内存分配器的数据结构。

MEMBLOCK 内存分配器采用逻辑框架如下图：

{% highlight bash %}
MEMBLOCK


                                         struct memblock_region
                       struct            +------+------+--------+------+
                       memblock_type     |      |      |        |      |
                       +----------+      | Reg0 | Reg1 | ...    | Regn |
                       |          |      |      |      |        |      |
                       | regions -|----->+------+------+--------+------+
                       | cnt      |      [memblock_memory_init_regions]
                       |          |
 struct           o--->+----------+
 memblock         |
 +-----------+    |
 |           |    |
 | memory   -|----o
 | reserved -|----o
 |           |    |                      struct memblock_region
 +-----------+    |    struct            +------+------+--------+------+
                  |    memblock_type     |      |      |        |      |
                  o--->+----------+      | Reg0 | Reg1 | ...    | Regn |
                       |          |      |      |      |        |      |
                       | regions -|----->+------+------+--------+------+
                       | cnt      |      [memblock_reserved_init_regions]
                       |          |
                       +----------+
{% endhighlight %}

从中可以看出 MEMBLOCK 采用了 3 层逻辑单元。第一层逻辑单元用于管理整个物理内存，
第二层逻辑单元管理不同类型的内存区，第三层逻辑单元管理独立的内存区块。
每层逻辑单元采用不同的数据结构进行维护，每种数据结构的相互配合，共同作为
MEMBLOCK 内存分配器的基础框架。

![](/assets/PDB/BiscuitOS/kernel/IND000100.png)

--------------------------------------------------------

###### <span id="第一层数据结构">第一层数据结构</span>

第一层逻辑单元用于维护系统的物理内存，采用数据结构 struct memblock 进行维护，
其定义如下：

{% highlight c %}
/**
 * struct memblock - memblock allocator metadata
 * @bottom_up: is bottom up direction?
 * @current_limit: physical address of the current allocation limit
 * @memory: usabe memory regions
 * @reserved: reserved memory regions
 * @physmem: all physical memory
 */
struct memblock {
	bool bottom_up;  /* is bottom up direction? */
	phys_addr_t current_limit;
	struct memblock_type memory;
	struct memblock_type reserved;
#ifdef CONFIG_HAVE_MEMBLOCK_PHYS_MAP
	struct memblock_type physmem;
#endif
};
{% endhighlight %}

struct memblock 数据结构从最顶层维护着整个 MEMBLOCK 分配器，各个成员对整个
分配器都起到了举足轻重的作用。

> bottom_up

成员 bottom_up 是一个 bool 变量，用于控制当 MEMBLOCK 分配器从顶部还是从
底部开始分配物理内存。如果该值为 true，那么 MEMBLOCK 会从底部到顶部的方向
分配内存，即 bottom-up; 如果该值为 false，那么 MEMBLOCK 会从顶部到底部的
方向分配内存，即 top-down.

> memory

memory 成员是一个 struct memblock_type 结构，这个成员用于指向系统可用物理
内存区，这个内存区维护着系统所有可用的物理内存，即系统 DRAM 对应的物理内存。

> reserved

reserved 成员是一个 struct memblock_type 结构，这个成员用于指向系统预留区，
也就是这个内存区的内存已经分配，在释放之前不能再次分配这个区内的内存区块。

> physmem

physmem 成员是一个 struct memblock_type 结构，这个成员用于指向系统物理映射
相关的内存区。这个内存区要使用必须打开宏 CONFIG_HAVE_MEMBLOCK_PHYS_MAP.

介绍完第一层采用的数据结构，接下来解析 MEMBLOCK 分配器采用这些数据结构 MEMBLOCK
分配器的第一层逻辑。内核被 uboot 加载到内存之后，并将 CPU 的执行权交给内核，
内核就开始构建内核的代码段，数据段等多种段，毕竟内核也是一个体积巨大的程序。
在这个阶段，与 MEMBLOCK 有关的操作如下：

###### <span id="创建section">创建 \_\_init_memblock，\_\_initdata_memblock section</span>

内核为 MEMBLOCK 内存分配器创建了一些私有的 section，这些 section 用于存放于
MEMBLOCK 分配器有关的函数和数据，这些 section 就是 \_\_init_memblock 和
\_\_initdata_memblock。\_\_init_memblock section 用于存储与 MEMBLOCK 分配器
相关的函数，\_\_initdata_memblock section 用于存储与 MEMBLOCK 分配器相关的
数据，定义如下：

{% highlight c %}
/* Used for MEMORY_HOTPLUG */
#define __meminit        __section(.meminit.text) __cold notrace __latent_entropy
#define __meminitdata    __section(.meminit.data)

#ifdef CONFIG_ARCH_DISCARD_MEMBLOCK
#define __init_memblock __meminit
#define __initdata_memblock __meminitdata
void memblock_discard(void);
#else
#define __init_memblock
#define __initdata_memblock
#endif
{% endhighlight %}

从上面的定义可知，如果启动 CONFIG_ARCH_DISCARD_MEMBLOCK 宏之后，内核启动到
后期，会将 \_\_init_memblock 和 __initdata_memblock 这两个 section 里面的内容
都丢弃，也就是系统正常运行之后不能再使用 MEMBLOCK 内存分配器。如果没有启用
CONFIG_ARCH_DISCARD_MEMBLOCK 宏，那么与 MEMBLOCK 内存分配器有关的函数都会
被加入到内核的代码段，与 MEMBLOCK 内存分配器相关的数据会被加入到内核的数据段。

###### 创建 struct memblock 实例

在创建完 \_\_init_memblock section 和 \_\_initdata_memblock section 之后，
MEMBLOCK 分配器开始创建 struct memblock 实例，这个实例此时作为最原始的 MEMBLOCK
分配器，描述了系统的物理内存的初始值，其代码如下

{% highlight c %}
struct memblock memblock __initdata_memblock = {
	.memory.regions		= memblock_memory_init_regions,
	.memory.cnt		= 1,	/* empty dummy entry */
	.memory.max		= INIT_MEMBLOCK_REGIONS,
	.memory.name		= "memory",

	.reserved.regions	= memblock_reserved_init_regions,
	.reserved.cnt		= 1,	/* empty dummy entry */
	.reserved.max		= INIT_MEMBLOCK_RESERVED_REGIONS,
	.reserved.name		= "reserved",

#ifdef CONFIG_HAVE_MEMBLOCK_PHYS_MAP
	.physmem.regions	= memblock_physmem_init_regions,
	.physmem.cnt		= 1,	/* empty dummy entry */
	.physmem.max		= INIT_PHYSMEM_REGIONS,
	.physmem.name		= "physmem",
#endif

	.bottom_up		= false,
	.current_limit		= MEMBLOCK_ALLOC_ANYWHERE,
};
{% endhighlight %}

内核定义了一个名为 memblock 的 struct memblock 实例做为 MEMBLOCK 分配器
的第一层逻辑实例。从定义可知，memblock 实例被放在 \_\_initdata_memblock section
内。定义之初，memblock 就指明了 memory，reserved，physeme 三个内存区的基础布局。
具体 memory, reserved, physeme 描述情况第二层数据结构。memblock 中还定义了
bottom_up 成员的值为 false，那么 MEMBLOCK 分配器默认采用 top-down 的方式，即
从顶部到底部的方向分配物理内存。current_limit 设置为 MEMBLOCK_ALLOC_ANYWHERE,
所以默认对 MEMBLOCK 分配器的范围不设置限制。

初始化完第一层逻辑之后，MEMBLOCK 分配器对内核初期的物理内存的维护就通过 memblock
实例展开。

![](/assets/PDB/BiscuitOS/kernel/IND000100.png)

--------------------------------------------------------

###### <span id="第二层数据结构">第二层数据结构</span>

第二层数据结构用于维护不同类型的内存区，采用数据结构 struct memblock_type 维护，
其源码如下：

{% highlight c %}
/**
 * struct memblock_type - collection of memory regions of certain type
 * @cnt: number of regions
 * @max: size of the allocated array
 * @total_size: size of all regions
 * @regions: array of regions
 * @name: the memory type symbolic name
 */
struct memblock_type {
	unsigned long cnt;
	unsigned long max;
	phys_addr_t total_size;
	struct memblock_region *regions;
	char *name;
};
{% endhighlight %}

struct memblock_type 数据结构维护了不同的内存区，由第一层逻辑结构可知，
MEMBLOCK 内存分配器一共维护 3 种不同的内存，分别为：memory (可用物理内存)，
reserved (预留内存)，physmem (物理映射内存)。3 类内存维护为 MEMBLOCK 内存
分配器实现内存的分配，回收，释放等操作提供了可能，其成员如下：

> cnt

该成员用于描述该类型内存区内含有多少个内存区块。这个成员有利于 MEMBLOCK 内存分
配器可以动态知道某种类型的内存区还有多少个内存区块。

> max

该成员用于描述该类型内存区最大可以含有多少个内存区块。当往某种类型的内存区添加
内存区块的时候，如果内存区的内存区块数超过 max 成员，那么 MEMBLOCK 内存分配器
就会增加内存区的容量，以此维护更多的内存区块。

> total_size

该成员用于统计内存区总共含有的物理内存数。

> regions

regions 成员是一个内存区块链表，用于维护属于这类型的所有内存区块。内存区块被维
护在这个链表上，并从地低址到高地址的方式排列。由于这个链表是一个数组构成的，所以
MEMBLOCK 分配器可以通过索引获得指定的内存区块。

> name

name 成员用于指明这个内存区的名字，MEMBLOCK 分配器目前支持的内存区名字有：
"memory", "reserved", "physmem"。

介绍完第二层采用的数据结构，接下来解析 MEMBLOCK 分配器采用这些数据结构 MEMBLOCK
分配器的第二层逻辑。因为第二层数据与第一层数据都在同一个 section 内，所以两者创
建的时间是一致的，所以第二层的数据也在 __init_memblock 或 __initdata_memblock
section 内，定义如下：

{% highlight c %}
static struct memblock_region memblock_memory_init_regions[INIT_MEMBLOCK_REGIONS] __initdata_memblock;
static struct memblock_region memblock_reserved_init_regions[INIT_MEMBLOCK_RESERVED_REGIONS] __initdata_memblock;
#ifdef CONFIG_HAVE_MEMBLOCK_PHYS_MAP
static struct memblock_region memblock_physmem_init_regions[INIT_PHYSMEM_REGIONS] __initdata_memblock;
#endif
{% endhighlight %}

从上面的定义可以知道, MEMBLOCK 内存分配器一共创建了三种内存区，使用三个数组维护，
每个数组的大小为 INIT_MEMBLOCK_REGIONS。从第一层逻辑构建时可以知道，第一层
memblock.memory 的 regions 成员指向了 memblock_memory_init_regions[] 数组，
而 memblock.reserved 的 regions 成员指向了 memblock_reserved_init_regions[]
数组。memblock.physmem 的 regions 成员指向 memblock_physmem_init_regions[]
数组。

###### memblock.memory 可用物理内存区

这个内存区用于管理系统可用的物理内存，其初始化时，cnt 成员为 1，max 成员设置为
INIT_MEMBLOCK_REGIONS， total_size 设置为 0. 名字设置为 "memory"。可用物理
内存指定是平台 DRAM 的体积。

###### memblock.reserved 预留区

这个内存区用于管理已经分配的内存区块，其初始化时，cnt 成员为 1，max 成员设置为
INIT_MEMBLOCK_RESERVED_REGIONS， total_size 设置为 0. 名字设置为 "reserved"。
每当 MEMBLOCK 分配器分配一段物理内存，这段物理内存就会被添加到预留区进行管理。

###### memblock.physmem 物理映射区

这个内存区用于物理映射，如果 CONFIG_HAVE_MEMBLOCK_PHYS_MAP 宏没有打开，那么
该内存区不使用，默认不使用。

对于第二层逻辑的内存区，MEMBLOCK 分配器提供了很多接口函数用于维护内存区，
基于这些接口，MEMBLOCK 分配器可以实现多种功能，其中包括：

> - 可用物理内存的分配和回收
>
> - 预留区内存的释放和分配
>
> - 内存区的状态管理
>
> - 内存区块的合并，插入和移除


![](/assets/PDB/BiscuitOS/kernel/IND000100.png)

--------------------------------------------------------

###### <span id="第三层数据结构">第三层数据结构</span>

第二层数据结构用于维护一块独立的内存区块，独立内存区块就是一定大小的物理内存区块，
采用数据结构 struct memblock_region 维护，其源码如下：

{% highlight c %}
/**
 * struct memblock_region - represents a memory region
 * @base: physical address of the region
 * @size: size of the region
 * @flags: memory region attributes
 * @nid: NUMA node id
 */
struct memblock_region {
	phys_addr_t base;
	phys_addr_t size;
	enum memblock_flags flags;
#ifdef CONFIG_HAVE_MEMBLOCK_NODE_MAP
	int nid;
#endif
};
{% endhighlight %}

内存区块用于维护一定长度的内存区块，从第二层逻辑上可以看出，内存区块可以
在不同的内存区之间移动，插入，合并等操作。内存区块具有不同的类型，不同类型
的内存区块不能合并，但同类型的内存区块可以合并。

> base

base 成员用于描述内存区块的起始物理地址。

> size

size 成员用于描述内存区块的长度。

> flags

flags 成员用于描述内存区块的类型。内存区块支持的类型是：热插拔型，mirror 型，
nomap 型，和普通型。其值定义为：

{% highlight c %}
/**
 * enum memblock_flags - definition of memory region attributes
 * @MEMBLOCK_NONE: no special request
 * @MEMBLOCK_HOTPLUG: hotpluggable region
 * @MEMBLOCK_MIRROR: mirrored region
 * @MEMBLOCK_NOMAP: don't add to kernel direct mapping
 */
enum memblock_flags {
	MEMBLOCK_NONE		= 0x0,	/* No special request */
	MEMBLOCK_HOTPLUG	= 0x1,	/* hotpluggable region */
	MEMBLOCK_MIRROR		= 0x2,	/* mirrored region */
	MEMBLOCK_NOMAP		= 0x4,	/* don't add to kernel direct mapping */
};
{% endhighlight %}

第三层逻辑单元不在第一二层逻辑单元初始化时进行创建，而在系统运行过程中，由
其他模块主动调用 MEMBLOCK 分配器的接口进行创建。MEMBLOCK 内存分配器为第三层
逻辑单元提供了很多接口，以此来管理所有的内存区块。这些接口基本分为以下几类：

> - 内存区块的分配
>
> - 内存区块的回收
>
> - 内存区块的合并和拆分
>
> - 内存区块状态管理

###### <span id="MEMBLOCK 内存分配器使用">MEMBLOCK 内存分配器使用</span>

MEMBLOCK 内存分配器的使用概述为：内核在启动初期，通过 MEMBLOCK 分配器获得
所需的物理内存，将某些物理内存添加到预留区，然后将使用完的物理内存退还给 MEMBLOCK
内存分配器等。所以本节讲解的是内核启动过程中，MEMBLOCK 内存分配器使用情况。

###### <span id="构建可用物理内存">构建可用物理内存</span>

内核在启动过程中，uboot 将平台具有的 DRAM 信息传递给 MEMBLOCK 分配器，MEMBLOCK
分配器将这些物理内存加入到可用物理内存区，其函数调用是：

{% highlight bash %}
start_kernel --->
    setup_arch --->
        setup_machine_fdt --->
            early_init_dt_scan_nodes --->
                early_init_dt_scan_memory
{% endhighlight %}

内核最后调用到 early_init_dt_scan_memory() 函数之后，从 DTS 中获得 DRAM 的
起始物理地址和长度，然后将其值传递给 early_init_dt_add_memory_arch()。该函数
对传入的物理地址和长度进行检测，如果检查通过，那么就调用 memblock_add() 函数
将这块物理内存区块加入到 memblock.memory 内存区内，这样系统关于 DRAM 的信息
就加入到 MEMBLOCK 分配器，接下来 MEMBLOCK 分配器就可以分配内存了。

通过上面的代码分析可以知道，DRAM 的信息通过 DTS 获得，所以可以配置 DTS 信息
来设置 DRAM 大小，具体如下：

由于本实践都是基于 ARM32 Vexpress V2P-CA9 平台，所以开发者在源码中打开
"arch/arm/boot/dts/vexpress-v2p-ca9.dts" 文件，找到如下节点：

{% highlight bash %}
memory@60000000 {
  device_type = "memory";
  reg = <0x60000000 0x40000000>;
};
{% endhighlight %}

这个节点用于描述平台 DRAM 的信息，device_type 属性值必须是 “memory”，reg
属性的 #address-cells 和 #size-cells 必须根据根节点定义进行编写，在上面的
例子中，根节点的 #address-cells 和 #size-cells 的值都是 1，所以 reg 属性
的第一个 u32 值代表起始物理地址，第二个 u32 值代表 DRAM 的长度。如果你使用
的平台 #address-cells 和 #size-cells 的值都是 2，那么 reg 属性的第一个和
第二个 u32 值构成了 DRAM 的起始物理地址，第三第四个 u32 值构成了 DRAM 的
长度。对于 DTS 的配置，请根据实际 DRAM 进行配置，就算这个配错了，uboot
阶段也会对 DTB 进行纠错，然后读出正确 DRAM 的尺寸。

###### <span id="adjust_lowmem_bounds">adjust_lowmem_bounds</span>

在构建完可用的物理内存之后，内核启动初期的物理内存分配都是通过 MEMBLOCK 分配器
进行，adjust_lowmem_bounds() 函数用于调整物理内存的基础信息，其源码分析如下：

{% highlight bash %}
start_kernel --->
    setup_arch --->
        adjust_lowmem_bounds
{% endhighlight %}

源码太长，分段解析：

{% highlight c %}
	for_each_memblock(memory, reg) {
		phys_addr_t block_start = reg->base;
		phys_addr_t block_end = reg->base + reg->size;

		if (reg->base < vmalloc_limit) {
			if (block_end > lowmem_limit)
				/*
				 * Compare as u64 to ensure vmalloc_limit does
				 * not get truncated. block_end should always
				 * fit in phys_addr_t so there should be no
				 * issue with assignment.
				 */
				lowmem_limit = min_t(u64,
							 vmalloc_limit,
							 block_end);

			/*
			 * Find the first non-pmd-aligned page, and point
			 * memblock_limit at it. This relies on rounding the
			 * limit down to be pmd-aligned, which happens at the
			 * end of this function.
			 *
			 * With this algorithm, the start or end of almost any
			 * bank can be non-pmd-aligned. The only exception is
			 * that the start of the bank 0 must be section-
			 * aligned, since otherwise memory would need to be
			 * allocated when mapping the start of bank 0, which
			 * occurs before any free memory is mapped.
			 */
			if (!memblock_limit) {
				if (!IS_ALIGNED(block_start, PMD_SIZE))
					memblock_limit = block_start;
				else if (!IS_ALIGNED(block_end, PMD_SIZE))
					memblock_limit = lowmem_limit;
			}

		}
	}
{% endhighlight %}

然后调用 for_each_memblock() 函数遍历系统内所有可用的物理内存，每遍历到
一块可用的物理内存区块，那么就计算出该内存区块的起始地址和终止地址，分别
存储到变量 block_start 和 block_end 内。接下来对地址进行进一步检查。
由于所有的可用物理内存区块在可用内存区链表内按从低地址到高地址排序，所以
函数首先检查变量到的可用内存区块的起始地址是否小于 VMALLOC 分配器可分配
的最小物理地址，如果是，那么就代表遍历到的可用物理区块和剩下未遍历到的可用
物理区块都可能与 VMALLOC 所能分配的物理内存重叠。对于不重叠的情况，这部分
代码不处理，直接跳过，但对于可能存在重叠的部分，函数继续做下面的检测。如果
遍历到的可用物理内存区块的终止物理地址最大低端物理地址，在这里，lowmem_limit
代表最大低端物理地址。lowmem_limit 变量第一次对比时值为 0，所以第一次遍历到
可用物理内存区块和 lowmem_limit 对比时，遍历到的可用物理内存区块的终止地址
比 lowmem_limit 大，所以进入 if 判断内执行。函数调用 min_t() 函数将
lowmem_limit 设置为 vmalloc_limit 和 block_end 中最小的一个，以此让
lowmem_limit 不会越过 VMALLOC 分配器可分配的最小物理地址。接下来查找第一个
不是按 PMD_SIZE 对齐的物理页，如果遍历到的可用物理内存区块的起始地址不是按
PMD_SIZE 方式对齐，那么将 memblock_limit 设置为 block_start；如果遍历到的
可用物理内存区块的起始地址是按 PMD_SIZE 方式对齐，但其终止地址不是按
PMD_SIZE 方式对齐，那么将 memblock_limit 设置为 lowmem_limit 的值。
最后再次循环可用物理内存区内剩下的内存区块。

{% highlight c %}
	arm_lowmem_limit = lowmem_limit;

	high_memory = __va(arm_lowmem_limit - 1) + 1;

	if (!memblock_limit)
		memblock_limit = arm_lowmem_limit;

	/*
	 * Round the memblock limit down to a pmd size.  This
	 * helps to ensure that we will allocate memory from the
	 * last full pmd, which should be mapped.
	 */
	memblock_limit = round_down(memblock_limit, PMD_SIZE);
{% endhighlight %}

接下来将计算出来的有效地址给全局变量进行赋值。首先将全局变量 arm_lowmem_limit
设置为 lowmem_limit，以此表示最大可分配的低端物理内存地址。Linux 内存布局将物理内存
分作了不同的区域用作不同的目的，其中 DRAM 中地址比较低的部分称为低端内存，这部分
内存通过线性的方式直接映射到虚拟内存，而 DRAM 中地址较高的部分称为高端内存，这部分
作为 VMALLOC 等其他分配器分配的物理内存，这部分物理内存和虚拟地址通过页表进行
映射。所以使用 arm_lowmem_limit 变量来划分 DRAM 的方位，小于 arm_lowmem_limit
的物理地址采用线性的方式直接映射到虚拟地址空间；大于 arm_lowmem_limit 的物理地址
采用页表映射到虚拟地址空间。high_memory 表示高端内存起始虚拟地址，其是
arm_lowmem_limit 物理地址对应的虚拟地址之后的一个地址。如果此时 memblock_limit
的值为空，那么将 arm_lowmem_limit 的值赋予 memblock_limit，这么做主要是因为
MEMBLOCK 分配器默认使用 Top-down 的方式从顶部往底部分配物理地址，所以有必要限定
一下最大可分配物理内存，不然会影响其他内存分配器。最后使用 round_down() 函数对
memblock_limit 变量做 PMD_SIZE 的向下对齐。

{% highlight c %}
	if (!IS_ENABLED(CONFIG_HIGHMEM) || cache_is_vipt_aliasing()) {
		if (memblock_end_of_DRAM() > arm_lowmem_limit) {
			phys_addr_t end = memblock_end_of_DRAM();

			pr_notice("Ignoring RAM at %pa-%pa\n",
				  &memblock_limit, &end);
			pr_notice("Consider using a HIGHMEM enabled kernel.\n");

			memblock_remove(memblock_limit, end - memblock_limit);
		}
	}

	memblock_set_current_limit(memblock_limit);
{% endhighlight %}

接下来，如果内核没有启动 CONFIG_HIGHMEM 宏，也就是内核没有启动高端内存，或者
cache_is_vipt_aliasing() 函数返回 ture，那么函数就执行下面一段代码。这段代码
首先获得 DRAM 的终止物理地址，如果终止物理地址比低端物理地址还大，那么 MEMBLOCK
分配器就要丢弃这部分内存，调用 memblock_remove() 函数然将 memblock_limit 开始
的物理地址到 [end - memblock_limit] 全部丢弃不能使用。处理完上面的代码之后，
函数就调用 memblock_set_current_limit() 函数将 MEMBLOCK 分配器的最大分配地址
设置为 memblock_limit (!注意，由于 MEMBLOCK 分配器默认采用 Top-down 自顶向下的
分配方式分配内存，如果改用 bottom-up 自底向上的分配方式，那么 MEMBLOCK 分配器
的限制也要做修改)。

###### <span id="arm_memblock_init">arm_memblock_init</span>

在初始化完基础的 MEMBLOCK 分配器之后，系统就开始使用 MEMBLOCK 分配器，首先是
函数 arm_memblock_init, 其调用逻辑是：

{% highlight bash %}
 start_kernel --->
     setup_arch --->
         arm_memblock_init
{% endhighlight %}

函数的主要作用就是通过 MEMBLOCK 分配器将指定的物理内存分配给特定功能。具体代码
如下：

{% highlight c %}
void __init arm_memblock_init(const struct machine_desc *mdesc)
{
	/* Register the kernel text, kernel data and initrd with memblock. */
	memblock_reserve(__pa(KERNEL_START), KERNEL_END - KERNEL_START);

	arm_initrd_init();

	arm_mm_memblock_reserve();

	/* reserve any platform specific memblock areas */
	if (mdesc->reserve)
		mdesc->reserve();

	early_init_fdt_reserve_self();
	early_init_fdt_scan_reserved_mem();

	/* reserve memory for DMA contiguous allocations */
	dma_contiguous_reserve(arm_dma_limit);

	arm_memblock_steal_permitted = false;
	memblock_dump_all();
}
{% endhighlight %}

函数首先调用 memblock_reserve() 函数将 kernel 对应的起始物理地址和终止物理地址
加入到预留区内，这样其他模块就不会影响到内核的正常工作。arm_initrd_init() 函数
将 INITRD 对应的物理内存区块加入到预留区。arm_mm_memblock_reserve() 函数将全局
页表对应的物理内存区块加入到预留区。接着，如果 mdesc-reserve 存在，也就是体系私有
的物理内存也会被加入到预留区内。然后调用 early_init_fdt_reserve_self() 和
early_init_fdt_scan_reserved_mem() 函数从 DTS 中获得需要预留的内存区块，并将
对应的物理内存加入到预留区。dma_contiguous_reserve() 函数将 DMA 需要预留的物理
内存加入到预留区内。最后将 arm_memblock_steal_permitted 设置为 false。通过
上面的操作，系统将要特殊使用的物理内存都添加到预留区内，之后 MEMBLOCK 分配器
就不会在分配这些物理内存。此时 MEMBLOCK 分配器的使用情况如下图：

{% highlight bash %}
MEMORY:   0x60000000 - 0xa0000000
Reserved: 0x60004000 - 0x60008000
Reserved: 0x60100000 - 0x60b90998
Reserved: 0x68000000 - 0x69d09c2e
Reserved: 0x9f000000 - 0xa0000000
{% endhighlight %}

上面提到，函数从 DTS 中获得需要预留的内存区，因此也可以通过 DTS 将某段物理内存
区段添加到预留区内，具体实践是基于 ARM32 Vexpress V2P-CA9 平台，所以开发者在源
码中打开 "arch/arm/boot/dts/vexpress-v2p-ca9.dts" 文件，找到如下节点：

{% highlight bash %}
  reserved-memory {
    #address-cells = <1>;
    #size-cells = <1>;
    ranges;

      Demo: Demo@0x82000000 {
        reg = <0x82000000 0x100000>;
        no-map;
      };

      Demo2: Demo@0x88000000 {
        reg = <0x88000000 0x100000>;
      };
  };
{% endhighlight %}

从上面的 DTS 可以看出 reserved-memory 节点下有两个子节点 Demo 和 Demo2，
其中 Demo 需要预留的物理内存区是 [0x82000000, 0x82100000]， Demo2 需要
预留的物理内存是 [0x88000000, 0x88100000]，但 Demo 子节点中包含了 no-map
属性，那么函数会将 Demo 需要的物理内存进行预留，但不会将其加入到 MEMBLOCK
分配器的预留区内。然而 Demo2 需要的物理内存 MEMBLOCK 分配器会从可用物理内存
中分配，并将其加入到预留内存，实际运行如下：

{% highlight bash %}
MEMORY:   0x60000000 - 0xa0000000
Reserved: 0x60004000 - 0x60008000
Reserved: 0x60100000 - 0x60b90998
Reserved: 0x68000000 - 0x69d09c2e
Reserved: 0x88000000 - 0x88100000
Reserved: 0x9f000000 - 0xa0000000
{% endhighlight %}

arm_memblock_init 函数运行完毕之后，内核需要的基础物理内存基本加入到 MEMBLOCK
分配器的预留区内，其他模块需要使用这些物理内存的时候就会分配失败。

###### <span id="bootmem_init">bootmem_init</span>

在接下来的启动过程中，就是各个模块使用 MEMBLOCK 分配器获得所需要的物理内存，
其中 bootmem_init() 函数用于构建内核不同 zone 做了前期准备，函数位置如下：

{% highlight bash %}
 start_kernel --->
     setup_arch --->
         paging_init --->
             bootmem_init
{% endhighlight %}

函数源码如下：

{% highlight c %}
void __init bootmem_init(void)
{
	unsigned long min, max_low, max_high;

	memblock_allow_resize();
	max_low = max_high = 0;

	find_limits(&min, &max_low, &max_high);

	early_memtest((phys_addr_t)min << PAGE_SHIFT,
		      (phys_addr_t)max_low << PAGE_SHIFT);

	/*
	 * Sparsemem tries to allocate bootmem in memory_present(),
	 * so must be done after the fixed reservations
	 */
	arm_memory_present();

	/*
	 * sparse_init() needs the bootmem allocator up and running.
	 */
	sparse_init();

	/*
	 * Now free the memory - free_area_init_node needs
	 * the sparse mem_map arrays initialized by sparse_init()
	 * for memmap_init_zone(), otherwise all PFNs are invalid.
	 */
	zone_sizes_init(min, max_low, max_high);

	/*
	 * This doesn't seem to be used by the Linux memory manager any
	 * more, but is used by ll_rw_block.  If we can get rid of it, we
	 * also get rid of some of the stuff above as well.
	 */
	min_low_pfn = min;
	max_low_pfn = max_low;
	max_pfn = max_high;
}
{% endhighlight %}

函数首先调用 memblock_allow_resize() 函数将 MEMBLOCK 分配器相关的全局
变量 memblock_can_resize 设置为 1，这样 MEMBLOCK 能够动态改变大小。接着
函数调用 find_limits() 函数，min 变量存储 DRAM 的起始页帧号，max_low 变量
存储 MEMBLOCK 可以分配的最大物理内存的页帧号，max_high 代表 DRAM 最大物理地址
的页帧号。然后通过三个值并调用 zone_sizes_init() 函数构建原始的 ZONE 框架。
构建完毕之后，将 min 赋值给全局变量 min_low_pfn，以此代表系统 DRAM 的起始物理
页帧号，同理将 max_low 赋值给 max_low_pfn 代表 MEMBLOCK 最大可以分配的物理
页帧号，最后将 max_high 赋值给 max_pfn 代表 DRAM 最大物理页帧号。

-----------------------------------------------------
# <span id="MEMBLOCK 调试"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000I.jpg)

#### MEMBLOCK 调试

MEMBLOCK 的调试分为启动阶段的调试和运行阶段的调试，对于启动阶段的调试，
MEMBLOCK 分配器默认提供了 debug 功能，debug 功能主要让开发者能够更便捷的获得 MEMBLOCK
分配器的信息，以及跟踪每次分配回收操作。开发者如果要开启 MEMBLOCK 的 debug 功能，需要
使用 CMDLINE 的方式传递给 kernel 参数 “memblock=debug”。在本实践平台上，开发者
可以通过两种方法开启这个功能，如下：

###### 通过 DTS 方式

DTS 的 chosen 节点就是用来传递 CMDLINE 参数给内核，所以可以子啊 chosen 根节点的
bootargs 属性中加入这个参数。由于实践是基于 ARM32 Vexpress V2P-CA9 平台，所以开发
者在源码中打开 "arch/arm/boot/dts/vexpress-v2p-ca9.dts" 文件，找到如下节点：

{% highlight bash %}
chosen {
        bootargs = "memblock=debug";
};
{% endhighlight %}

###### 通过 uboot 传入方式

Uboot 加载 kernel 完毕之后，会向 Uboot 传入启动参数，所以开发者可以在 uboot 传入
的参数列表中加入这个参数。对于本实践平台，开发者只需修改
"BiscuitOS/output/linux-5.x/RunQemuKernel.sh" 文件即可，参数如下修改：

{% highlight bash %}
do_running()
{
	${QEMUT} -M vexpress-a9 -m ${MEM_SIZE}M -kernel ${ROOT}/linux/linux/arch/${ARCH}/boot/zImage -dtb ${ROOT}/linux/linux/arch/${ARCH}/boot/dts/vexpress-v2p-ca9.dtb -nodefaults -serial stdio -nographic -append "earlycon root=/dev/ram0 rw rootfstype=ext4 console=ttyAMA0 init=/linuxrc loglevel=8 memblock=debug" -initrd ${ROOT}/ramdisk.img
}
{% endhighlight %}

###### debug 模式下的内核

当开发者打开 MEMBLOCK 的 debug 功能之后，内核启动信息中就会出现更多有用的 debug
信息。如下：

{% highlight bash %}
Booting Linux on physical CPU 0x0
Linux version 5.0.0 (buddy@biscuitos) (gcc version 7.4.1 20181213 [linaro-7.4-2019.02 revision 56ec6f6b99cc167ff0c2f8e1a2eed33b1edc85d4] (Linaro GCC 7.4-2019.02)) #54 SMP Sat Mar 16 09:27:54 CST 2019
CPU: ARMv7 Processor [410fc090] revision 0 (ARMv7), cr=10c5387d
CPU: PIPT / VIPT nonaliasing data cache, VIPT nonaliasing instruction cache
OF: fdt: Machine model: V2P-CA9
Region: [0x64400000 - 0xa0000000]
Region: [0x64100000 - 0x64300000]
Region: [0x60000000 - 0x64000000]
Malformed early option 'earlycon'
Memory policy: Data cache writeback
memblock_reserve: [0x60100000-0x60b90997] arm_memblock_init+0x30/0x160
memblock_reserve: [0x68000000-0x69cf6fff] arm_memblock_init+0xec/0x160
memblock_reserve: [0x60004000-0x60007fff] arm_memblock_init+0x108/0x160
memblock_reserve: [0x69cf7000-0x69d02c2d] arm_memblock_init+0x11c/0x160
memblock_remove: [0x4c000000-0x4c7fffff] __fdt_scan_reserved_mem+0x260/0x2dc
Reserved memory: created DMA memory pool at 0x4c000000, size 8 MiB
OF: reserved mem: initialized node vram@4c000000, compatible id shared-dma-pool
memblock_reserve: [0x9f000000-0x9fffffff] memblock_alloc_range+0x54/0x6c
cma: Reserved 16 MiB at 0x9f000000
MEMBLOCK configuration:
 memory size = 0x40000000 reserved size = 0x037975c6
 memory.cnt  = 0x1
 memory[0x0]	[0x60000000-0x9fffffff], 0x40000000 bytes flags: 0x0
 reserved.cnt  = 0x5
 reserved[0x0]	[0x60004000-0x60007fff], 0x00004000 bytes flags: 0x0
 reserved[0x1]	[0x60100000-0x60b90997], 0x00a90998 bytes flags: 0x0
 reserved[0x2]	[0x64000000-0x640fffff], 0x00100000 bytes flags: 0x0
 reserved[0x3]	[0x68000000-0x69d02c2d], 0x01d02c2e bytes flags: 0x0
 reserved[0x4]	[0x9f000000-0x9fffffff], 0x01000000 bytes flags: 0x0
memblock_reserve: [0x9effffd8-0x9effffff] memblock_alloc_base_nid+0x50/0x68
memblock_reserve: [0x9effe000-0x9effefff] memblock_alloc_base_nid+0x50/0x68
memblock_reserve: [0x9effd000-0x9effdfff] memblock_alloc_base_nid+0x50/0x68
memblock_reserve: [0x9effc000-0x9effcfff] memblock_alloc_base_nid+0x50/0x68
memblock_reserve: [0x9effb000-0x9effbfff] memblock_alloc_base_nid+0x50/0x68
memblock_reserve: [0x9effa000-0x9effafff] memblock_alloc_base_nid+0x50/0x68
memblock_reserve: [0x9eff9000-0x9eff9fff] memblock_alloc_base_nid+0x50/0x68
memblock_reserve: [0x9eff8000-0x9eff8fff] memblock_alloc_base_nid+0x50/0x68
memblock_reserve: [0x9eff7000-0x9eff7fff] memblock_alloc_base_nid+0x50/0x68
memblock_reserve: [0x9eff4000-0x9eff5fff] memblock_alloc_base_nid+0x50/0x68
memblock_reserve: [0x9eff6000-0x9eff6fff] memblock_alloc_base_nid+0x50/0x68
memblock_reserve: [0x9eff3000-0x9eff3fff] memblock_alloc_base_nid+0x50/0x68
memblock_reserve: [0x9eff2000-0x9eff2fff] memblock_alloc_base_nid+0x50/0x68
On node 0 totalpages: 262144
memblock_alloc_try_nid_nopanic: 8388608 bytes align=0x40 nid=0 from=0x00000000 max_addr=0x00000000 alloc_node_mem_map.constprop.10+0x68/0xa4
memblock_reserve: [0x9e7f2000-0x9eff1fff] memblock_alloc_internal+0x120/0x1a8
  Normal zone: 2048 pages used for memmap
  Normal zone: 0 pages reserved
  Normal zone: 262144 pages, LIFO batch:63
memblock_alloc_try_nid_nopanic: 128 bytes align=0x40 nid=0 from=0x00000000 max_addr=0x00000000 setup_usemap.constprop.14+0x5c/0x68
{% endhighlight %}

![](/assets/PDB/BiscuitOS/kernel/IND000100.png)

----------------------------------

#### MEMBLOCK 用户空间调试

Linux 也将 MEMBLOCK 的调试信息添加到 debugfs 子系统里，所以在系统正常运行之后，
开发者也可以通过下面的办法读取 MEMBLOCK 分配器的信息：

{% highlight bash %}
cd /sys/kernel/debug/memblock

# 查看可用物理内存
cat memory
   0: 0x60000000..0x9fffffff

# 查看预留内存
cat reserved
  0: 0x60004000..0x60007fff
  1: 0x60100000..0x60b90997
  2: 0x64000000..0x640fffff
  3: 0x68000000..0x69d02c2d
  4: 0x9e6db000..0x9e7e6fff
  5: 0x9e7e96a0..0x9effefff
  6: 0x9efff080..0x9efff0f7
  7: 0x9efff100..0x9efff403
  8: 0x9efff440..0x9efff784
  9: 0x9efff7c0..0x9efff7fb
  10: 0x9efff800..0x9efff983
  11: 0x9efff9c0..0x9efffb84
  12: 0x9efffbc0..0x9efffc4f
  13: 0x9efffc80..0x9efffc8f
  14: 0x9efffcc0..0x9efffcc3
  15: 0x9efffd00..0x9efffd62
  16: 0x9efffd80..0x9efffde2
  17: 0x9efffe00..0x9efffe62
  18: 0x9efffe80..0x9efffe83
  19: 0x9efffe94..0x9efffec6
  20: 0x9efffec8..0x9efffee2
  21: 0x9efffee4..0x9efffefe
  22: 0x9effff00..0x9effff1f
  23: 0x9effff24..0x9effff3e
  24: 0x9effff40..0x9fffffff
{% endhighlight %}

开发者也可以通过如下命令查看系统物理内存布局：

{% highlight bash %}
cat /proc/iomem

10000008-1000000b : dat
10000048-1000004b : dat
1000004c-1000004f : dat
100000a0-100000ab : vexpress-syscfg.6.auto
10002000-10002fff : i2c@2000
10004000-10004fff : aaci@4000
  10004000-10004fff : aaci-pl041
10005000-10005fff : mmci@5000
  10005000-10005fff : mmci@5000
10006000-10006fff : kmi@6000
  10006000-10006fff : kmi-pl050
10007000-10007fff : kmi@7000
  10007000-10007fff : kmi-pl050
10009000-10009fff : uart@9000
  10009000-10009fff : uart@9000
1000a000-1000afff : uart@a000
  1000a000-1000afff : uart@a000
1000b000-1000bfff : uart@b000
  1000b000-1000bfff : uart@b000
1000c000-1000cfff : uart@c000
  1000c000-1000cfff : uart@c000
10011000-10011fff : timer@11000
10012000-10012fff : timer@12000
10016000-10016fff : i2c@16000
10017000-10017fff : rtc@17000
  10017000-10017fff : rtc-pl031
1001f000-1001ffff : clcd@1f000
10020000-10020fff : clcd@10020000
  10020000-10020fff : clcd@10020000
40000000-43ffffff : flash@0,00000000
44000000-47ffffff : flash@0,00000000
48000000-49ffffff : psram@2,00000000
60000000-9fffffff : System RAM
  60008000-609fffff : Kernel code
  60b00000-60b90997 : Kernel data
{% endhighlight %}


-----------------------------------------------------
# <span id="内存分配器进阶实践"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000F.jpg)

#### MEMBLOCK 进阶研究

> - [MEMBLOCK 内存块合并的分析与实践](#MEMBLOCK 内存块合并的分析与实践)
>
> - [MEMBLOCK 内存块拆分的分析与实践](#MEMBLOCK 内存块拆分的分析与实践)

--------------------------------------

<span id="MEMBLOCK 内存块合并的分析与实践"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000B.jpg)

#### MEMBLOCK 内存块合并的分析与实践

MEMBLOCK 内存分配器将物理内存分作的可用物理内存和预留内存，不同物理内存维护在 MEMBLOCK
分配器的不同链表上。每当往特定内存区中加入内存块的时候，就会出现新加入的内存块和原先存在
内存区中的内存区块存在不同的位置关系，比如相邻，相交，重叠，包含的位置关系。对于这些关系，
MEMBLOCK 分配器采用了一定的算法将这些内存块进行合并或新建等操作。本节研究的主题就是
MEMBLOCK 分配器如何将一块新的内存区块添加到内存区。

###### 预备知识：位置关系

MEMBLOCK 分配器将特定内存区内的所用内存区块都按低地址到高地址的排序，这样的策略就会出现
新加入的内存块会和已存在的内存块位置上存在前后关系。例如下面两种关系图(注：rbase,rend
为已存在的内存区块；end,base 为新加入的内存区块)：

{% highlight bash %}
1） rbase > end

 base                    end        rbase               rend
 +-----------------------+          +-------------------+
 |                       |          |                   |
 | New region            |          | Exist regions     |
 |                       |          |                   |
 +-----------------------+          +-------------------+

2）rbase == endi

                         rbase                      rend
                        | <----------------------> |
 +----------------------+--------------------------+
 |                      |                          |
 | New region           | Exist regions            |
 |                      |                          |
 +----------------------+--------------------------+
 | <------------------> |
 base                   end
{% endhighlight %}

从上面的逻辑图可知，新加入的内存区块位于已存在内存区块的前面。类似的新加入的
内存区块位于已存在内存区块的后面。

###### 预备知识：种类

MEMBLOCK 分配器将内存区块分为了以下几种类型：

{% highlight c %}
/**
 * enum memblock_flags - definition of memory region attributes
 * @MEMBLOCK_NONE: no special request
 * @MEMBLOCK_HOTPLUG: hotpluggable region
 * @MEMBLOCK_MIRROR: mirrored region
 * @MEMBLOCK_NOMAP: don't add to kernel direct mapping
 */
enum memblock_flags {
        MEMBLOCK_NONE           = 0x0,  /* No special request */
        MEMBLOCK_HOTPLUG        = 0x1,  /* hotpluggable region */
        MEMBLOCK_MIRROR         = 0x2,  /* mirrored region */
        MEMBLOCK_NOMAP          = 0x4,  /* don't add to kernel direct mapping */
};
{% endhighlight %}

从上面的定义可以看出，内存区块可以分为：普通内存区块，热插拔内存区块，镜像内存区块，
不能映射的内存区块。由于这样的策略，就会导致当新加入的内存区块与已存在的内存区块如果存在
包含或交叉的部分，两者的类型不同就不能进行合并。如下图逻辑图：

{% highlight bash %}
                 rbase     Exist regions        rend
                 | <--------------------------> |
 +---------------+--------+---------------------+
 |               |        |                     |
 |               |        |                     |
 |               |        |                     |
 +---------------+--------+---------------------+
 | <--------------------> |
 base   New region        end

 rbase                     rend
 | <---------------------> |
 +----------------+--------+----------------------+
 |                |        |                      |
 | Exist regions  |        |                      |
 |                |        |                      |
 +----------------+--------+----------------------+
                  | <---------------------------> |
                  base      new region            end
{% endhighlight %}

##### 实践

基于上面的分析，开发者可以通过一个实际的例子来认知这个问题，以及 MEMBLOCK 如何
处理这个问题，具体实践请看:

> - [MEMBLOCK 内存块合并的分析与实践之 memblock_reserve](/blog/MMU-ARM32-MEMBLOCK-memblock_reserve/)

--------------------------------------

<span id="MEMBLOCK 内存块拆分的分析与实践"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000K.jpg)

#### MEMBLOCK 内存块拆分的分析与实践

MEMBLOCK 内存分配器将物理内存分作的可用物理内存和预留内存，不同物理内存维护在 MEMBLOCK
分配器的不同链表上。当 MEMBLOCK 分配器执行释放，移除操作的时候，实际上从不同的内存区链表
中移除指定长度的内存区块。这样的操作就会引入一个问题，如果移除的部分正好位于某块内存区块内存，
在移除只有，原始的内存区块就会被拆分做两块新的内存区块，对于上述遇到的问题，本节重点研究
MEMBLOCK 分配器的拆分问题。

###### 实践

基于上面的介绍，开发者可以通过一个实际的例子来认知这个问题，以及 MEMBLOCK 如何
处理这个问题，具体实践请看：

> - [MEMBLOCK 内存块拆分的分析与实践之 memblock_remove](/blog/MMU-ARM32-MEMBLOCK-memblock_remove/)

![](/assets/PDB/BiscuitOS/kernel/IND000100.png)

-----------------------------------------------------

<span id="MEMBLOCK_API-LIST"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000H.jpg)

#### MEMBLOCK API List

> [for_each_free_mem_range](/blog/MMU-ARM32-MEMBLOCK-for_each_free_mem_range/)
>
> [for_each_free_mem_range_reverse](/blog/MMU-ARM32-MEMBLOCK-for_each_free_mem_range_reverse/)
>
> [for_each_mem_range](/blog/MMU-ARM32-MEMBLOCK-for_each_mem_range/)
>
> [for_each_mem_range_rev](/blog/MMU-ARM32-MEMBLOCK-for_each_mem_range_rev/)
>
> [for_each_reserved_mem_region](/blog/MMU-ARM32-MEMBLOCK-for_each_reserved_mem_region/)
>
> [memblock_add](/blog/MMU-ARM32-MEMBLOCK-memblock_add/)
>
> [memblock_add_node](/blog/MMU-ARM32-MEMBLOCK-memblock_add_node/)
>
> [memblock_add_range](/blog/MMU-ARM32-MEMBLOCK-memblock_add_range)
>
> [__memblock_alloc_base](/blog/MMU-ARM32-MEMBLOCK-__memblock_alloc_base/)
>
> [memblock_alloc_base](/blog/MMU-ARM32-MEMBLOCK-memblock_alloc_base/)
>
> [memblock_alloc_base_nid](/blog/MMU-ARM32-MEMBLOCK-memblock_alloc_base_nid/)
>
> [memblock_alloc_range](/blog/MMU-ARM32-MEMBLOCK-memblock_alloc_range/)
>
> [memblock_bottom_up](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> [memblock_end_of_DRAM](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> [memblock_find_in_range](/blog/MMU-ARM32-MEMBLOCK-memblock_find_in_range/)
>
> [memblock_find_in_range_node](/blog/MMU-ARM32-MEMBLOCK-memblock_find_in_range_node/)
>
> [memblock_free](/blog/MMU-ARM32-MEMBLOCK-memblock_free/)
>
> [memblock_get_current_limit](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> [memblock_get_region_node](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> [memblock_is_hotpluggable](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> [memblock_is_memory](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> [memblock_is_mirror](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> [memblock_is_nomap](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> [memblock_is_region_memory](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> [memblock_is_region_reserved](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> [memblock_phys_alloc_nid](/blog/MMU-ARM32-MEMBLOCK-memblock_phys_alloc_nid/)
>
> [memblock_phys_alloc_try_nid](/blog/MMU-ARM32-MEMBLOCK-memblock_phys_alloc_try_nid/)
>
> [memblock_phys_mem_size](/blog/MMU-ARM32-MEMBLOCK-memblock_phys_mem_size/)
>
> [memblock_remove](/blog/MMU-ARM32-MEMBLOCK-memblock_remove/)
>
> [memblock_reserve](/blog/MMU-ARM32-MEMBLOCK-memblock_reserve/)
>
> [memblock_reserved_size](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> [memblock_set_current_limit](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> [memblock_set_region_node](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> [memblock_start_of_DRAM](/blog/MMU-ARM32-MEMBLOCK-memblock_information/)
>
> [__next_mem_range](/blog/MMU-ARM32-MEMBLOCK-__next_mem_range/)
>
> [__next_mem_range_rev](/blog/MMU-ARM32-MEMBLOCK-__next_mem_range_rev/)
>
> [__next_reserved_mem_region](/blog/MMU-ARM32-MEMBLOCK-__next_reserved_mem_region/)

-----------------------------------------------

# <span id="附录">附录</span>

> [MEMBLOCK 内存分配器](/blog/MMU-ARM32-MEMBLOCK-index/)
>
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

