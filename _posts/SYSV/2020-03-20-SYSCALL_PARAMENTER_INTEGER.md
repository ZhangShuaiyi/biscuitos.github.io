---
layout: post
title:  "SYSCALL: 添加整形参数的系统调用"
date:   2019-11-27 08:35:30 +0800
categories: [HW]
excerpt: syscall.
tags:
  - syscall
---

![](/assets/PDB/BiscuitOS/kernel/IND00000L0.PNG)

> Email: BuddyZhang1 <buddy.zhang@aliyun.com>

#### 目录

> - [实践原理](#B10)
>
> - [实践准备](#B11)
>
> - [添加用户空间实现](#B14)
>
> - [添加内核系统调用入口](#B12)
>
> - [添加内核实现](#B13)
>
> - [运行系统调用](#B15)

![](/assets/PDB/BiscuitOS/kernel/IND000100.png)

--------------------------------------------

#### <span id="B10">实践原理</span>

用户空间可用通过系统调用可以将一个或多个整形数据传递给内核空间。对于
整形参数，用户空间只是传递整形值到内核空间，因此内核部分的系统调用函数
可以利用自己的函数堆栈暂存，因此内核空间和用户空间不必进行跨区拷贝操作。
系统调用参数支持的整形包括了: char, unsigned char, short, unsigend short,
int, unsigned int, long, unsigned long, long long, unsigned long long 等。
内核空间对应的系统调用函数可以按下面的方式定义参数:

{% highlight c %}
SYSCALL_DEFINE6(hello_BiscuitOS,
                        bool, enable,
                        char, integer_char,
                        short, integer_short,
                        int, integer_int,
                        unsigned long, integer_long,
                        unsigned long long, integer_llong)
{
	...
}
{% endhighlight %}

用户空间对应的系统调用，只需传递函数形参或整形常量即可，使用如下:

{% highlight c %}
int main(void)
{
        unsigned long long integer_llong = 0x12345678abcdef88;
        unsigned long integer_long = 0x98765432;
        int integer_int = 0x12345678;
        short integer_short = 0x1234;
        char integer_char = 'B';
        bool enable = true;
        int ret;

        /*
         * sys_hello_BiscuitOS: Integer paramenter
         * kernel:
         *       SYSCALL_DEFINE6(hello_BiscuitOS,
         *                       bool, enable,
         *                       char, integer_char,
         *                       short, integer_short,
         *                       int, integer_int,
         *                       unsigned long, integer_long,
         *                       unsigned long long, integer_llong);
         */
        ret = syscall(__NR_hello_BiscuitOS,
                                        enable,
                                        integer_char,
                                        integer_short,
                                        integer_int,
                                        integer_long,
                                        integer_llong);
	return 0;
}
{% endhighlight %}

对于系统调用传递多个整形参数的方法，开发者可以参考如下文档:

> - [添加零个参数的系统调用](/blog/SYSCALL_PARAMENTER_ZERO/)
>
> - [添加一个参数的系统调用](/blog/SYSCALL_PARAMENTER_ONE/)
>
> - [添加两个参数的系统调用](/blog/SYSCALL_PARAMENTER_TWO/)
>
> - [添加三个参数的系统调用](/blog/SYSCALL_PARAMENTER_THREE/)
>
> - [添加四个参数的系统调用](/blog/SYSCALL_PARAMENTER_FOUR/)
>
> - [添加五个参数的系统调用](/blog/SYSCALL_PARAMENTER_FIVE/)
>
> - [添加六个参数的系统调用](/blog/SYSCALL_PARAMENTER_SIX/)

对于系统调用的返回值，内核会返回一个整形值，至于整形值的含义，开发者
可以根据需求进行返回.

![](/assets/PDB/BiscuitOS/kernel/IND000100.png)

-------------------------------------------

#### <span id="B11">实践准备</span>

BiscuitOS 目前支持 6 大平台进行实践，本文以 ARM32 为例子进行讲解，如果
开发者需要在其他平台实践，可以参考下面文档进行实践:

> - [ARM32 架构中添加一个新的系统调用](/blog/SYSCALL_ADD_NEW_ARM/)
>
> - [ARM64 架构中添加一个新的系统调用](/blog/SYSCALL_ADD_NEW_ARM64/)
>
> - [i386 架构中添加一个新的系统调用](/blog/SYSCALL_ADD_NEW_I386/)
>
> - [X86_64 架构中添一个新的系统调用](/blog/SYSCALL_ADD_NEW_X86_64/)
>
> - [RISCV32 架构中添一个新的系统调用](/blog/SYSCALL_ADD_NEW_RISCV32/)
>
> - [RISCV64 架构中添加一个新的系统调用](/blog/SYSCALL_ADD_NEW_RISCV64/)

本实践基于 ARM32 架构，因此在实践之前需要准备一个 ARM32 架构的运行
平台，开发者可以在 BiscuitOS 进行实践，如果还没有搭建 BiscuitOS
ARM32 实践环境的开发者，可以参考如下文档进行搭建:

> - [BiscuitOS 上搭建 ARM32 实践环境](/blog/Linux-5.0-arm32-Usermanual/)

开发环境搭建完毕之后，可以继续下面的内容，如果开发者不想采用
BiscuitOS 提供的开发环境，可以继续参考下面的内容在开发者使用
的环境中进行实践。(推荐使用 BiscuitOS 开发环境)。搭建完毕之后，
使用如下命令:

{% highlight bash %}
cd BiscuitOS/
make linux-5.0-arm32_defconfig
make
{% endhighlight %}

![](/assets/PDB/RPI/RPI000339.png)

上图显示了 ARM32 实践环境的位置，以及相关的 README.md 文档，开发者
可以参考 README.md 的内容搭建一个运行在 QEMU 上的 ARM32 Linux 开发
环境:

![](/assets/PDB/RPI/RPI000340.png)

![](/assets/PDB/BiscuitOS/kernel/IND000100.png)

--------------------------------------------

#### <span id="B14">添加用户空间实现</span>

BiscuitOS 提供了一套完整的系统调用编译系统，开发者可以使用下面步骤部署一个
简单的用户空间调用接口文件，BiscuitOS 并可以对该文件进行交叉编译，安装，
打包和目标系统上运行的功能，节省了很多开发时间。如果开发者不想使用这套
编译机制，可以参考下面的内容进行移植。开发者首先获得用户空间系统调用
基础源码，如下:

{% highlight bash %}
cd BiscuitOS
make linux-5.0-arm32_defconfig
make menuconfig
{% endhighlight %}

![](/assets/PDB/RPI/RPI000341.png)

选择并进入 "[\*] Package  --->"

![](/assets/PDB/RPI/RPI000342.png)

选择 "[\*]   strace" 和 "[\*]   System Call" 并进入 "[\*]   System Call  --->"

![](/assets/PDB/RPI/RPI000343.png)

选择并进入 "[\*]   sys_hello_BiscuitOS  --->"

![](/assets/PDB/RPI/RPI000471.png)

选择 "[\*]   Syscall paramenter: Ineger  --->" 保存配置并退出. 
接下来执行下面的命令部署用户空间系统调用程序部署:

{% highlight bash %}
cd BiscuitOS
make
{% endhighlight %}

![](/assets/PDB/RPI/RPI000339.png)

执行完毕后，终端输出相关的信息, 接下来进入源码位置，使用如下命令:

{% highlight bash %}
cd BiscuitOS/output/linux-5.0-arm32/package/SYSCALL_Integer_common-0.0.1
{% endhighlight %}

这个目录就是用于部署用户空间系统调用程序，开发者继续使用命令:

{% highlight bash %}
cd BiscuitOS/output/linux-5.0-arm32/package/SYSCALL_Integer_common-0.0.1
make prepare
make download
{% endhighlight %}

执行上面的命令之后，BiscuitOS 自动部署了程序所需的所有文件，如下:

{% highlight bash %}
tree
{% endhighlight %}

![](/assets/PDB/RPI/RPI000472.png)

上图中，main.c 与用户空间系统调用相关的源码,
"SYSCALL_Integer_common-0.0.1/Makefile" 是 main.c 交叉编译的逻辑。
"SYSCALL_Integer_common-0.0.1/BiscuitOS_syscall.c" 文件是新系统调用
内核实现。因此对于用户空间的系统调用，开发者只需关注 main.c, 内容如下:

![](/assets/PDB/RPI/RPI000473.png)

根据在内核中创建的入口，这里定义了入口宏的值为 400，一定要与内核定义
的入口值相呼应. 在上图的程序中，定义了多个类型的整形变量。然后将这些
变量通过 "syscall()" 函数触发系统调用，传入的第一个参数是 
"\_\_NR_hello_BiscuitOS", 其为系统调用号，该值为 400. 传入的第二个参数
是一个 bool 整形变量，第三个传入的是一个 char 型变量，占用一个字节。
第三个传入的值是一个 short 变量，占用 2 个字节。第四个传入的值是一个
int 变量，占用 4 个字节。第五个传入的是一个 long 型变量，占用 4 个字节，
最后传入的是一个 long long 型变量，占用 8 个字节。源码准备好之后，接
下来是交叉编译源码并打包到 rootfs 里，最后在 ARM32 上运行。使用如下命令:

{% highlight bash %}
cd BiscuitOS/output/linux-5.0-arm32/package/SYSCALL_Integer_common-0.0.1
make
make install
make pack
{% endhighlight %}

![](/assets/PDB/BiscuitOS/kernel/IND000100.png)

--------------------------------------------

#### <span id="B12">添加内核系统调用入口</span>

ARM32 架构提供了便捷的方法在内核中添加一个新的系统调用入口。
开发者修改内核源码下 "arch/arm/tools/syscall.tbl" 文件，在
该文件的底部添加信息如下:

![](/assets/PDB/RPI/RPI000310.png)

如上面内容所示，在文件最后一行添加了名为 hello_BiscuitOS 的
系统调用，400 代表系统调用号，hello_BiscuitOS 为系统调用的
名字，sys_hello_BiscuitOS 为系统调用在内核的实现。至此系统
号已经添加完毕。

![](/assets/PDB/BiscuitOS/kernel/IND000100.png)

--------------------------------------------

#### <span id="B13">添加内核实现</span>

添加完系统号之后，需要在内核中添加系统调用的具体实现。开发者
可以参考下面的例子进行添加。本例子在内核源码 "fs/" 目录下添加
一个名为 BiscuitOS_syscall.c 的文件，如下:

{% highlight bash %}
cp -rfa BiscuitOS/output/linux-5.0-arm32/package/SYSCALL_Integer_common-0.0.1/BiscuitOS_syscall.c  BiscuitOS/output/linux-5.0-arm32/linux/linux/fs/
cd BiscuitOS/output/linux-5.0-arm32/linux/linux/fs
vi BiscuitOS_syscall.c
{% endhighlight %}

![](/assets/PDB/RPI/RPI000474.png)

由于要介绍从用户空间传递的 6 个参数，因此使用 SYSCALL_DEFINE6 宏来定义，
与用户空间相对应，第一个参数是系统调用的名字 "hello_BiscuitOS"。第二个
参数是 bool 类型，第三个是 char 型，第四个是 short 型，第五个是 int 型，
第六个是 long 型，最后一个是 long long 型变量。函数接受这些参数之后，
直接在内核空间打印，不涉及任何拷贝操作。准备好源码之后，接着修改内核源
码 "fs/Kconfig" 文件，添加如下内容:

{% highlight bash %}
cd BiscuitOS/output/linux-5.0-arm32/linux/linux/fs
vi Kconfig
{% endhighlight %}

![](/assets/PDB/RPI/RPI000333.png)

接着修改内核源码 "fs/Makefile" 文件，添加内容如下:

{% highlight bash %}
cd BiscuitOS/output/linux-5.0-arm32/linux/linux/fs
vi Makefile
{% endhighlight %}

![](/assets/PDB/RPI/RPI000334.png)

接着是配置内核，将 BiscuitOS_syscall.c 文件加入内核编译树，如下:

{% highlight bash %}
cd BiscuitOS/output/linux-5.0-arm32/linux/linux/
make menuconfig ARCH=arm
{% endhighlight %}

![](/assets/PDB/RPI/RPI000335.png)

选择并进入 "File systems  --->"

![](/assets/PDB/RPI/RPI000336.png)

选择 "\[*] BiscuitOS syscall hello" 并保存内核配置。

接着重新编译内核。

{% highlight bash %}
cd BiscuitOS/output/linux-5.0-arm32/linux/linux/
make ARCH=arm CROSS_COMPILE=BiscuitOS/output/linux-5.0-arm32/arm-linux-gnueabi/arm-linux-gnueabi/bin/arm-linux-gnueabi- -j4
{% endhighlight %}

编译内核中会打印相关的信息如下图:

![](/assets/PDB/RPI/RPI000306.png)

从上面的编译信息可以看出，之前的修改已经生效。编译系统调用相关的脚本
自动为hello_BiscuitOS 生成了相关的系统调用，

![](/assets/PDB/BiscuitOS/kernel/IND000100.png)

--------------------------------------------

#### <span id="B15">运行系统调用</span>

在一切准备好之后，最后一步就是在 ARM32 上运行系统调用，参考下面
命令进行运行:

{% highlight bash %}
cd BiscuitOS/output/linux-5.0-arm32/
./RunBiscuitOS.sh
{% endhighlight %}

![](/assets/PDB/RPI/RPI000475.png)

从运行结果可以看出，内核接收到用户空间传递下来的整形参数。可以使用 strace 
工具查看具体的系统调用过程，如下:

{% highlight bash %}
~ #
~ # strace SYSCALL_Integer_common-0.0.1
{% endhighlight %}

![](/assets/PDB/RPI/RPI000476.png)

从 strace 打印的消息可以看出 "syscall_0x190()" 正好程序里产生的系统调用.

![](/assets/PDB/BiscuitOS/kernel/IND000100.png)

-----------------------------------------------

#### <span id="C0">附录</span>

> [BiscuitOS Home](https://biscuitos.github.io/)
>
> [BiscuitOS Driver](/blog/BiscuitOS_Catalogue/)
>
> [Linux Kernel](https://www.kernel.org/)
>
> [Bootlin: Elixir Cross Referencer](https://elixir.bootlin.com/linux/latest/source)
>

#### 捐赠一下吧 🙂

![MMU](/assets/PDB/BiscuitOS/kernel/HAB000036.jpg)
