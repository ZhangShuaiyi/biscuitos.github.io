---
layout: post
title:  "统计二叉树叶子数"
date:   2019-05-12 05:30:30 +0800
categories: [HW]
excerpt: TREE 统计二叉树叶子数.
tags:
  - Tree
---

![DTS](/assets/PDB/BiscuitOS/kernel/IND00000B.jpg)

> [Github: 统计二叉树叶子数](https://github.com/BiscuitOS/HardStack/tree/master/Algorithem/tree/binary-tree/Leaf)
>
> Email: BuddyZhang1 <buddy.zhang@aliyun.com>


# 目录

> - [原理分析](#原理分析)
>
> - [实践](#实践)
>
> - [附录](#附录)

-----------------------------------

# <span id="原理分析">原理分析</span>

![DTS](/assets/PDB/BiscuitOS/boot/BOOT000071.png)

上图是一棵完美的二叉树，树中非叶子节点的度都是 2 (即每个非叶子的节点都有两个孩子)。
叶子节点的度都是 0，因此可以遍历节点中所有度为 0 的节点数。因此，如果使用递归的办法
统计所有叶子的数，那么可以参考如下实现：

{% highlight ruby %}
/* Leaf counter */
static int BinaryTree_LeafCount(struct binary_node *node)
{
	static int count;

	if (node != NULL) {
		if (node->left == NULL && node->right == NULL)
			count++;

		BinaryTree_LeafCount(node->left);
		BinaryTree_LeafCount(node->right);
	}
	return count;
}
{% endhighlight %}

在上面的代码中，如果遍历的节点为 NULL，那么直接返回；如果不为 NULL，那么就查看
当前节点是否存在孩子，如果不存在，那么就将局部静态变量加 1 操作，以此统计
叶子的数。测试中提供了一套二叉树先序的数据:

{% highlight ruby %}
static int Perfect_BinaryTree_data[] = {
                                  200, 143, 754, 6, -1, -1, 3, -1, -1,
                                  386, 7, -1, -1, 9, -1, -1, 876, 486,
                                  8, -1, -1, 2, -1, -1, 740, 1, -1, -1,
                                  5, -1, -1 };
{% endhighlight %}

从这组数据中，可以完整的看到先序在二叉树的体现。接下来通过一个实践代码进一步
认识二叉树叶子的统计过程。

--------------------------------------------------

# <span id="实践">实践</span>

> - [实践源码](#实践源码)
>
> - [源码编译](#源码编译)
>
> - [源码运行](#源码运行)
>
> - [运行分析](#运行分析)

#### <span id="实践源码">实践源码</span>

> [实践源码 GitHub](https://github.com/BiscuitOS/HardStack/blob/master/Algorithem/tree/binary-tree/Leaf/binary.c)

开发者也可以使用如下命令获得：

{% highlight ruby %}
wget https://raw.githubusercontent.com/BiscuitOS/HardStack/master/Algorithem/tree/binary-tree/Leaf/binary.c
{% endhighlight %}

实践源码具体内容如下：

{% highlight c %}
/*
 * Binary-Tree.
 *
 * (C) 2019.05.12 <buddy.zhang@aliyun.com>
 *
 * This program is free software; you can redistribute it and/or modify
 * it under the terms of the GNU General Public License version 2 as
 * published by the Free Software Foundation.
 */
#include <stdio.h>
#include <stdlib.h>

/* binary-tree node */
struct binary_node {
	int idx;
	struct binary_node *left;
	struct binary_node *right;
};

/* Perfect Binary Tree
 *                               (200)
 *                                 |
 *                 o---------------+---------------o
 *                 |                               |
 *               (143)                           (876)
 *                 |                               |
 *          o------+-----o                  o------+-----o
 *          |            |                  |            |
 *        (754)        (386)              (486)        (740)
 *          |            |                  |            |
 *      o---+---o    o---+---o          o---+---o    o---+---o
 *      |       |    |       |          |       |    |       |
 *     (6)     (3)  (7)     (9)        (8)     (2)  (1)     (5)
 */
static int Perfect_BinaryTree_data[] = {
                                  200, 143, 754, 6, -1, -1, 3, -1, -1,
                                  386, 7, -1, -1, 9, -1, -1, 876, 486,
                                  8, -1, -1, 2, -1, -1, 740, 1, -1, -1,
                                  5, -1, -1 };

static int counter = 0;
static int *BinaryTree_data = Perfect_BinaryTree_data;

/* Preorder Create Binary-tree */
static struct binary_node *Preorder_Create_BinaryTree(struct binary_node *node)
{
	int ch = BinaryTree_data[counter++];

	/* input from terminal */
	if (ch == -1) {
		return NULL;
	} else {
		node =
		   (struct binary_node *)malloc(sizeof(struct binary_node));
		node->idx = ch;

		/* Create left child */
		node->left  = Preorder_Create_BinaryTree(node->left);
		/* Create right child */
		node->right = Preorder_Create_BinaryTree(node->right);
		return node;
	}
}

/* Leaf counter */
static int BinaryTree_LeafCount(struct binary_node *node)
{
	static int count;

	if (node != NULL) {
		if (node->left == NULL && node->right == NULL)
			count++;

		BinaryTree_LeafCount(node->left);
		BinaryTree_LeafCount(node->right);
	}
	return count;
}

/* Post-Free BinaryTree */
static void Postorder_Free_BinaryTree(struct binary_node *node)
{
	if (node == NULL) {
		return;
	} else {
		Postorder_Free_BinaryTree(node->left);
		Postorder_Free_BinaryTree(node->right);
		free(node);
		node = NULL;
	}
}

int main()
{
	/* Define binary-tree root */
	struct binary_node *BiscuitOS_root;

	printf("Preorder Create BinaryTree\n");
	BiscuitOS_root = Preorder_Create_BinaryTree(BiscuitOS_root);


	/* The leaf number for Binary-Tree */
	printf("The Binary-Tree leaf: %d\n",
				BinaryTree_LeafCount(BiscuitOS_root));

	/* Postorder free binary-tree */
	Postorder_Free_BinaryTree(BiscuitOS_root);

	return 0;
}
{% endhighlight %}

--------------------------------------

#### <span id="源码编译">源码编译</span>

将实践源码保存为 binary.c，然后使用如下命令进行编译：

{% highlight ruby %}
gcc bianry.c -o binary
{% endhighlight %}

--------------------------------------

#### <span id="源码运行">源码运行</span>

实践源码的运行很简单，可以使用如下命令，并且运行结果如下：

{% highlight ruby %}
binary-tree/Leaf$ ./binary
Preorder Create BinaryTree
The Binary-Tree leaf: 8
{% endhighlight %}

--------------------------------------

#### <span id="运行分析">运行分析</span>

从运行的结果可以看出，获得叶子的数量和设计的一致。

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
