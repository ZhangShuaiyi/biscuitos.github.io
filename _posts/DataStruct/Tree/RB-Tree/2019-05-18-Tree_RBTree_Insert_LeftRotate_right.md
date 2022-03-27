---
layout: post
title:  "红黑树左旋：父节点是祖父的右孩子，引起的右旋转"
date:   2019-05-18 05:30:30 +0800
categories: [HW]
excerpt: TREE 红黑树左旋：父节点是祖父的右孩子，引起的右旋转.
tags:
  - Tree
---

![DTS](/assets/PDB/BiscuitOS/kernel/IND00000R.jpg)

> [Github: 插入一个红节点引起左旋转](https://github.com/BiscuitOS/HardStack/tree/master/Algorithem/tree/rb-tree/Insert/Case5)
>
> Email: BuddyZhang1 <buddy.zhang@aliyun.com>

# 目录

> - [红黑树插入一个红节点引起左旋转原理](#原理分析)
>
> - [红黑树插入一个红节点引起左旋转与 2-3 树的关系](#23Tree)
>
> - [红黑树插入一个红节点引起左旋转实践](#实践)
>
> - [附录](#附录)

-----------------------------------
<span id="原理分析"></span>

![DTS](/assets/PDB/BiscuitOS/boot/BOOT100000.gif)

对结点 E 做左旋操作时，其右孩子为 S 而不是 T.nil，那么以 E 到 S 的链为
"支轴" 进行。使 S 成为该子树新的根结点，E 成为 S 的左孩子，E 的左孩子成为 S 的
右孩子.

![DTS](/assets/PDB/BiscuitOS/boot/BOOT000075.png)

如上图，当插入 6 之后，红黑树 5 节点需要进行左旋达到平衡，那么以 4 到 5 的链为
"支轴" 进行。使用 5 节点称为 6 的新的根节点，4 称为 5 的左孩子，6 称为 5 的右
孩子。如下图：

![DTS](/assets/PDB/BiscuitOS/boot/BOOT000076.png)

##### 核心代码实现

为了实现左旋操作，参考内核中的实现进行分析，位于 lib/rbtree.c 文件中，关于左旋的
实现如下：

{% highlight ruby %}
		tmp = gparent->rb_right;
		if (parent == tmp) { /* parent == gparent->rb_right */
			tmp = parent->rb_left;

			gparent->rb_right = tmp;
			parent->rb_left = gparent;
			if (tmp)
				rb_set_parent_color(tmp, gparent, RB_BLACK);
			__rb_rotate_set_parents(gparent, parent, root, RB_RED);
			break;
		}
{% endhighlight %}

核心代码首先判断 gparent (gparent 为 parent 的父节点) 的右孩子是否存在，对于需要左
旋的部分，gparent 的右孩子是存在的。接着按照左旋的原理，以 gparent 节点到 parent 节
点为支轴进行左旋。此时 parent 的左孩子变成了 gparent 的右孩子，对应的代码就是：
"gparent->rb_right = tmp", gparent 自己变成了 parent 的左孩子, 对应的代码就是：
"parent->rb_left = gparent"。如果此时 tmp 存在，也就是原先 parent 的左孩子存在，
那么，设置 tmp 的父节点为 gparent。接着调用 __rb_rotate_set_parents() 函数修改
gparent 和 parent 之间的关系，代码如下：

{% highlight ruby %}
static inline void
__rb_rotate_set_parents(struct rb_node *old, struct rb_node *new,
		       struct rb_root *root, int color)
{
	struct rb_node *parent = rb_parent(old);
	new->__rb_parent_color = old->__rb_parent_color;
	rb_set_parent_color(old, new, color);
	__rb_change_child(old, new, parent, root);
}
{% endhighlight %}

首先获得 gparent 的父节点，然后将 parent 在红黑树 __rb_parent_color 成员继承
gparent 的 __rb_parent_color. 然后设置 gparent 的 __rb_parent_color 为
parent。最后修改 gparent 原先的父节点的节点信息，函数如下：

{% highlight ruby %}
static inline void
__rb_change_child(struct rb_node *old, struct rb_node *new,
		  struct rb_node *parent, struct rb_root *root)
{
	if (parent) {
		if (parent->rb_left == old)
			parent->rb_left = new;
		else
			parent->rb_right = new;
	} else
		root->rb_node = new;
}
{% endhighlight %}

逻辑很简单，就是判断 gpraent 是原始父节点的左孩子还是右孩子，然后将原始父节点的
左孩子或右孩子指向 parent 节点。如果 gparent 的父节点不存在，那么 gparent 原先是
root 节点，那么就将 root 节点指向 parent 节点。

通过上面的源码，rbtree 已经完成左旋操作，并设置好了各个节点之间的关系，使红黑树再一次
达到平衡。

-----------------------------------
<span id="23Tree"></span>

![DTS](/assets/PDB/BiscuitOS/kernel/IND00000Q.jpg)

### 红黑树插入一个红节点引起右旋转与 2-3 树的关系

毕竟红黑树是 2-3 树的一种表现形式，因此插入一个红节点到引起红黑树左旋转的原理也符合 2-3 树
的原理。由于父节点是祖父节点的右孩子，那么祖父节点是一个 2- 节点。父节点此时是一个红节点，
在向其右侧添加一个红节点，那么此时父节点和子节点都是红节点，与 p0 构成了一个临时的 4- 节点，
此时两个红节点不符合 2-3 树的要求，对此处进行分裂，分裂的逻辑就是提起中间的节点，将 4- 节点
分裂成三个 2- 节点，由于提取的节点 n1 需要向上融合，所以需要将其设置为红色，但如果 n1 已经
是根节点，那么 n1 设置为黑色，形成一个 2- 节点，从红黑树角度来看就是一次左旋，如下图。

![](/assets/PDB/BiscuitOS/boot/BOOT000095.png)

更多红黑树与 2-3 树的关系请看文档：

> [红黑树与 2-3 树的关系分析](/blog/Tree_2-3-tree/)

--------------------------------------------------
<span id="实践"></span>

![DTS](/assets/PDB/BiscuitOS/kernel/IND00000D.jpg)

# 红黑树插入一个红节点引起右旋转实践

> - [实践源码](#实践源码)
>
> - [源码编译](#源码编译)
>
> - [源码运行](#源码运行)
>
> - [运行分析](#运行分析)

#### <span id="实践源码">实践源码</span>

> [实践源码 GitHub](https://github.com/BiscuitOS/HardStack/tree/master/Algorithem/tree/rb-tree/Insert/Case5)

开发者可以从上面的链接中获得所有的实践代码，也可以使用如下命令获得：

{% highlight ruby %}
mkdir -p Insert_ROOT
cd Insert_ROOT
wget https://raw.githubusercontent.com/BiscuitOS/HardStack/master/Algorithem/tree/rb-tree/Insert/Case5/Makefile
wget https://raw.githubusercontent.com/BiscuitOS/HardStack/master/Algorithem/tree/rb-tree/Insert/Case5/README.md
wget https://raw.githubusercontent.com/BiscuitOS/HardStack/master/Algorithem/tree/rb-tree/Insert/Case5/rb_run.c
wget https://raw.githubusercontent.com/BiscuitOS/HardStack/master/Algorithem/tree/rb-tree/Insert/Case5/rbtree.c
wget https://raw.githubusercontent.com/BiscuitOS/HardStack/master/Algorithem/tree/rb-tree/Insert/Case5/rbtree.h
{% endhighlight %}

实践源码具体内容如下：

{% highlight c %}
/*
 * RB-Tree Manual.
 *
 * (C) 2019.05.14 <buddy.zhang@aliyun.com>
 *
 * This program is free software; you can redistribute it and/or modify
 * it under the terms of the GNU General Public License version 2 as
 * published by the Free Software Foundation.
 */
#include <stdio.h>
#include <stdlib.h>

/* rbtree */
#include <rbtree.h>

struct node {
	struct rb_node node;
	unsigned long runtime;
};

/* n points to a rb_node */
#define node_entry(n) container_of(n, struct node, node)

static struct node node0 = { .runtime = 0x20 };
static struct node node1 = { .runtime = 0x40 };
static struct node node2 = { .runtime = 0x50 };

/* rbroot */
static struct rb_root BiscuitOS_rb = RB_ROOT;

/* Insert private node into rbtree */
static int rbtree_insert(struct rb_root *root, struct node *node)
{
	struct rb_node **new = &(root->rb_node), *parent = NULL;

	/* Figure out where to put new node */
	while (*new) {
		struct node *this = node_entry(*new);
		int result;

		/* Compare runtime */
		result = this->runtime - node->runtime;

		/* setup parent */
		parent = *new;

		/*
		 *        (this)
		 *         /  \
		 *        /    \
		 *  (little)   (big)
		 *
		 */
		if (result < 0)
			new = &((*new)->rb_right);
		else if (result > 0)
			new = &((*new)->rb_left);
		else
			return 0;
	}

	/* Add new node and rebalance tree */
	rb_link_node(&node->node, parent, new);
	rb_insert_color(&node->node, root);
}

static int count = 20;
/* Middle-order iterate over RB tree */
static void Middorder_IterateOver(struct rb_node *node)
{
	if (!node) {
		return;
	} else {
		Middorder_IterateOver(node->rb_left);
		printf("%#lx ", node_entry(node)->runtime);
		Middorder_IterateOver(node->rb_right);
	}
}

int main()
{
	struct node *np;

	/* Insert rb_node */
	rbtree_insert(&BiscuitOS_rb, &node0);
	rbtree_insert(&BiscuitOS_rb, &node1);
	rbtree_insert(&BiscuitOS_rb, &node2);
	Middorder_IterateOver(BiscuitOS_rb.rb_node);
	printf("\n");

	return 0;
}
{% endhighlight %}

--------------------------------------

#### <span id="源码编译">源码编译</span>

使用如下命令进行编译：

{% highlight ruby %}
make
{% endhighlight %}

--------------------------------------

#### <span id="源码运行">源码运行</span>

实践源码的运行很简单，可以使用如下命令，并且运行结果如下：

{% highlight ruby %}
rb-tree/Insert$ ./rbtree
0x20 0x40 0x50
{% endhighlight %}

--------------------------------------

#### <span id="运行分析">运行分析</span>

在实践代码中，使用二叉查找树的方法，向红黑树插入红节点，然后调用 rb_insert_color()
红黑树实际插入操作。开发者可以使用调试工具跟踪 rb_insert_color() 的调用过程。

-----------------------------------------------

# <span id="附录">附录</span>

> [Data Structure Visualizations](https://www.cs.usfca.edu/~galles/visualization/Algorithms.html)
>
> [红黑树与 2-3 树的关系](https://www.jianshu.com/p/57a0329b2801)
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
