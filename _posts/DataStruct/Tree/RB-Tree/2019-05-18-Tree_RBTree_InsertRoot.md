---
layout: post
title:  "红黑树插入操作之：插入根节点"
date:   2019-05-18 05:30:30 +0800
categories: [HW]
excerpt: TREE 红黑树插入操作之：插入根节点.
tags:
  - Tree
---

![DTS](/assets/PDB/BiscuitOS/kernel/IND00000R.jpg)

> [Github: 红黑树插入操作之：插入根节点](https://github.com/BiscuitOS/HardStack/tree/master/Algorithem/tree/rb-tree/Insert/Case0)
>
> Email: BuddyZhang1 <buddy.zhang@aliyun.com>


# 目录

> - [红黑树插入根节点原理](#原理分析)
>
> - [红黑树插入根节点与 2-3 树的关系](#23Tree)
>
> - [红黑树插入根节点实践](#实践)
>
>
> - [附录](#附录)

-----------------------------------
<span id="原理分析"></span>

![DTS](/assets/PDB/BiscuitOS/kernel/IND00000Y.jpg)

# 红黑树插入根节点

向一棵空的红黑树插入根节点是比较简单，只需将红黑树的根节点指向该节点，并且设置根节点的
颜色为黑色。

###### 核心代码实现

{% highlight ruby %}

static inline void
__rb_insert(struct rb_node *node, struct rb_root *root,
	    bool newleft, struct rb_node **leftmost,
	    void (*augment_rotate)(struct rb_node *old, struct rb_node *new))
{
	struct rb_node *parent = rb_red_parent(node), *gparent, *tmp;

	if (newleft)
		*leftmost = node;

	while (true) {
		/*
		 * Loop invariant: node is red.
		 */
		if (!parent) {
			/*
			 * The inserted node is root. Either this is the
			 * first node, or we recursed at Case 1 below and
			 * are no longer violating 4).
			 */
			rb_set_parent_color(node, NULL, RB_BLACK);
			break;
		}
	}
}
{% endhighlight %}

当调用 __rb_insert() 函数向红黑树插入一个节点时候，如果检查到该节点没有 parent 节点，
那么这个节点就是根节点，然后调用 rb_set_parent_color() 函数将该节点的颜色标记为黑色。

-----------------------------------
<span id="23Tree"></span>

![DTS](/assets/PDB/BiscuitOS/kernel/IND00000Q.jpg)

# 红黑树插入根节点与 2-3 树的关系

毕竟红黑树是 2-3 树的一种表现形式，因此插入根节点的原理也符合 2-3 树的原理。当插入一个
根节点的时候，此时红黑树中不存在任何 2- 或者 3- 的节点，因此不用考虑融合和提取，只需
之间插入到红黑树作为一个 2- 节点。

![](/assets/PDB/BiscuitOS/boot/BOOT000091.png)

更多红黑树与 2-3 树的关系请看文档：

> [红黑树与 2-3 树的关系分析](https://biscuitos.github.io/blog/Tree_2-3-tree/)

--------------------------------------------------
<span id="实践"></span>

![DTS](/assets/PDB/BiscuitOS/kernel/IND00000D.jpg)

# 红黑树插入根节点实践

> - [实践源码](#实践源码)
>
> - [源码编译](#源码编译)
>
> - [源码运行](#源码运行)
>
> - [运行分析](#运行分析)

#### <span id="实践源码">实践源码</span>

> [实践源码 GitHub](https://github.com/BiscuitOS/HardStack/tree/master/Algorithem/tree/rb-tree/Insert/Case0)

开发者可以从上面的链接中获得所有的实践代码，也可以使用如下命令获得：

{% highlight ruby %}
mkdir -p Insert_ROOT
cd Insert_ROOT
wget https://raw.githubusercontent.com/BiscuitOS/HardStack/master/Algorithem/tree/rb-tree/Insert/Case0/Makefile
wget https://raw.githubusercontent.com/BiscuitOS/HardStack/master/Algorithem/tree/rb-tree/Insert/Case0/README.md
wget https://raw.githubusercontent.com/BiscuitOS/HardStack/master/Algorithem/tree/rb-tree/Insert/Case0/rb_run.c
wget https://raw.githubusercontent.com/BiscuitOS/HardStack/master/Algorithem/tree/rb-tree/Insert/Case0/rbtree.c
wget https://raw.githubusercontent.com/BiscuitOS/HardStack/master/Algorithem/tree/rb-tree/Insert/Case0/rbtree.h
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
rb-tree/Insert/Case0$ ./rbtree
0x20
{% endhighlight %}

--------------------------------------

#### <span id="运行分析">运行分析</span>

在实践代码中，使用二叉查找树的方法，向红黑树中插入根节点，然后调用 rb_insert_color()
红黑树实际插入操作。开发者可以使用调试工具跟踪 rb_insert_color() 的调用过程。

-----------------------------------------------

# <span id="附录">附录</span>

> [Data Structure Visualizations](https://www.cs.usfca.edu/~galles/visualization/Algorithms.html)
>
> [红黑树与 2-3 树的关系](https://www.jianshu.com/p/57a0329b2801)
>
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
