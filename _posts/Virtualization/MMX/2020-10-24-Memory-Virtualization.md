---
layout: post
title:  "内存虚拟化: Memory Virtualization base on KVM"
date:   2020-10-24 06:00:00 +0800
categories: [HW]
excerpt: Memory Virtualization.
tags:
  - KVM
---

![](/assets/PDB/BiscuitOS/kernel/IND00000L0.PNG)

![](/assets/PDB/RPI/RPI100100.png)

#### 目录

> - [内存虚拟化基础](#A)

![](/assets/PDB/BiscuitOS/kernel/IND000100.png)

----------------------------------

<span id="A"></span>

![](/assets/PDB/BiscuitOS/kernel/IND00000T.jpg)

#### 内存虚拟化基础


#### 基础概念

-------------------------------------

###### GVA 

![](/assets/PDB/HK/HK000611.png)

在虚拟化中，虚拟机称为 Guest OS, 其运行在物理机的 VMX Non-Root 模式下。在 Guest OS 内部，当 Guest OS 进入保护模式之后，Guest OS 内部程序和系统可以直接访问的地址称为 GVA。GVA 全称 "Guest Vritual Address"，即虚拟机程序运行的虚拟地址。GVA 的逻辑含义与物理机的虚拟地址概念一样，都是通过页表的方式与物理内存想关联，只是在 Guest OS 内部，GVA 通过页表与 GPA 相关联, 这里的 GPA 就是 Guest OS 的物理地址。

-------------------------------------

###### GPA/GFN

![](/assets/PDB/HK/HK000612.png)

在虚拟化中，虚拟机称为 Guest OS，其运行在物理机的 VMX Non-Root 模式下。在 VMX Non-Root 模式下，Guest OS 可以访问的物理内存对应的地址称为 GPA. GPA 全称 "Guest Physical Address", 因此 GPA 就称为虚拟机的物理地址. 从 Guest OS 内部来看，GPA 构成的地址空间就是 Guest OS 的物理内存，而从 Host 角度来看，GPA 构成的地址空间则是 Host 端的虚拟地址空间.

Guest OS 将物理内存按 PAGE_SIZE 大小将物理内存划分成一个个独立的物理内存区域，然后从 0 地址到高地址的方式，为每个物理内存区块进行编号，这个号码称为 Guest OS 的物理页帧号，简称 "GFN", GFN 与 GPA 的关系如下:

> GFN = GPA \>\> PAGE_SIZE

-------------------------------------

###### HVA

![](/assets/PDB/HK/HK000613.png)

在虚拟化中，Host OS 的虚拟地址称为 HVA. 虚拟机对于 Host OS 来说是一个个独立的 qemu-kvm 进程，Guest OS 内部运行在 VMX Non-root 模式，Guest OS 是无法感知到 Host 虚拟地址的存在，只能感知到 Guest OS 内部的物理地址存在; 相反 Host OS 也无法感知到 Guest OS 的物理地址存在，对 Host OS 来说都是虚拟地址。qemu-kvm 在创建 Guest OS 的时候，从 Host OS 上申请一段段虚拟地址空间作为 Guest OS 的物理内存空间，这里的 Host 虚拟内存可能充当 Guest OS 的物理内存、SMIO、E820 预留区等。

---------------------------------------

###### HPA/HFN/PFN

![](/assets/PDB/HK/HK000614.png)

在虚拟化中，HPA 称为物理主机的物理地址。与通常概念的物理地址一样，这是物理机正真的物理内存提供的地址。物理内存提供了物理地址、物理页帧号和 struct page 等信息。Host OS 直接访问虚拟地址，硬件 MMU/TLB 等设备会自动查询页表，将一个 HVA 自动转换成 HPA; 如果页表不存在，那么 Host OS 可以通过缺页机制为 HVA 与某个 HGA 建立映射。

Host OS 按 PAGE_SIZE 的大小将物理内存划分为多个内存区域，并从 0 地址到高地址的循序为每个物理内存区域进行编号，这个号码称为页帧号，也称为 PFN, 或者 HFN. 三者之间的关系如下:

> PFN = HPA \>\> PAGE_SIZE

![](/assets/PDB/BiscuitOS/kernel/IND000100.png)

-----------------------------------------------

#### <span id="Z0">附录</span>

> [BiscuitOS Home](https://biscuitos.github.io/)
>
> [BiscuitOS Blog 2.0](https://biscuitos.github.io/blog/BiscuitOS_Catalogue/)
>
> [Linux Kernel](https://www.kernel.org/)
>
> [Bootlin: Elixir Cross Referencer](https://elixir.bootlin.com/linux/latest/source)
>

#### 捐赠一下吧 🙂

![MMU](/assets/PDB/BiscuitOS/kernel/HAB000036.jpg)
