---
layout: post
title:  "后序创建二叉树"
date:   2019-05-12 05:30:30 +0800
categories: [HW]
excerpt: TREE 后序创建二叉树.
tags:
  - Tree
---

![DTS](/assets/PDB/BiscuitOS/kernel/IND00000S.jpg)

> [Github: 后序创建二叉树](https://github.com/BiscuitOS/HardStack/tree/master/Algorithem/tree/binary-tree/Create/Postorder)
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
二叉树的后序指的是：先左孩子，然后右孩子，最后根。例如上面的二叉树，后序的结果就是：
6 3 754 7 9 386 143 8 2 486 1 5 740 876 200。因此，如果使用递归加后序的方法创建
二叉树的话，那么可以参考如下实现：

{% highlight ruby %}
/* Preorder Create Binary-tree
 *
 *   Don't direct Postorder create binary-tree, but we can find the last
 *   node is root, and previous node is right-child for root, and pre-previous
 *   node is left-child for root..... So, we can create binary tree like this.
 */
static struct binary_node *Postorder_Create_BinaryTree(struct binary_node *node)
{
	int ch = BinaryTree_data[--counter];

	/* input from terminal */
	if (ch == -1) {
		return NULL;
	} else {
		node=
		   (struct binary_node *)malloc(sizeof(struct binary_node));
		node->idx = ch;

		/* Create right child */
		node->right  = Postorder_Create_BinaryTree(node->right);
		/* Create left child */
		node->left = Postorder_Create_BinaryTree(node->left);
		return node;
	}
}
{% endhighlight %}

在上面的代码中，如果 ch 的为 -1 代表该节点不存在，如果使用后序的方法，先创建右孩子，
再创建左孩子，最后创建根节点，那样是行不通的，直接导致程序跑偏。开发者进一步观察
后序节点的排列关系，可以发现，最后一个节点就是根节点，倒数第二个就是根节点的右孩子，
倒数第三个节点就是根节点的左孩子，以此类推，可以从根节点，再到右节点，最后到左节点的
方式构建，正如上面的代码。为了构建一棵二叉树，代码中提前准备了一组后序的数据，如下:

{% highlight ruby %}
static int Perfect_BinaryTree_data[] = {
                                   -1, -1, 6, -1, -1, 3, 754, -1, -1, 7,
                                   -1, -1, 9, 386, 143, -1, -1, 8, -1, -1,
                                   2, 486, -1, -1, 1, -1, -1, 5, 740, 876,
                                   200 };
{% endhighlight %}

从这组数据中，可以完整的看到后序在二叉树的体现。接下来通过一个实践代码进一步
认识后序创建二叉树的过程。

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

> [实践源码 binar.c on GitHub](https://github.com/BiscuitOS/HardStack/blob/master/Algorithem/tree/binary-tree/Create/Postorder/binary.c)

开发者也可以使用如下命令获得：

{% highlight ruby %}
wget https://raw.githubusercontent.com/BiscuitOS/HardStack/master/Algorithem/tree/binary-tree/Create/Postorder/binary.c
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
                                   -1, -1, 6, -1, -1, 3, 754, -1, -1, 7,
                                   -1, -1, 9, 386, 143, -1, -1, 8, -1, -1,
                                   2, 486, -1, -1, 1, -1, -1, 5, 740, 876,
                                   200 };
/* Must indicate the number of counter on Postorder Create binary-tree */
static int counter = 31;
static int *BinaryTree_data = Perfect_BinaryTree_data;

/* Preorder Create Binary-tree
 *
 *   Don't direct Postorder create binary-tree, but we can find the last
 *   node is root, and previous node is right-child for root, and pre-previous
 *   node is left-child for root..... So, we can create binary tree like this.
 */
static struct binary_node *Postorder_Create_BinaryTree(struct binary_node *node)
{
	int ch = BinaryTree_data[--counter];

	/* input from terminal */
	if (ch == -1) {
		return NULL;
	} else {
		node=
		   (struct binary_node *)malloc(sizeof(struct binary_node));
		node->idx = ch;

		/* Create right child */
		node->right  = Postorder_Create_BinaryTree(node->right);
		/* Create left child */
		node->left = Postorder_Create_BinaryTree(node->left);
		return node;
	}
}

/* Pre-Traverse Binary-Tree */
static void Preorder_Traverse_BinaryTree(struct binary_node *node)
{
	if (node == NULL) {
		return;
	} else {
		printf("%d ", node->idx);
		/* Traverse left child */
		Preorder_Traverse_BinaryTree(node->left);
		/* Traverse right child */
		Preorder_Traverse_BinaryTree(node->right);
	}
}

/* Midd-Traverse Binary-Tree */
static void Middorder_Traverse_BinaryTree(struct binary_node *node)
{
	if (node == NULL) {
		return;
	} else {
		Middorder_Traverse_BinaryTree(node->left);
		printf("%d ", node->idx);
		Middorder_Traverse_BinaryTree(node->right);
	}
}

/* Post-Traverse Binary-Tree */
static void Postorder_Traverse_BinaryTree(struct binary_node *node)
{
	if (node == NULL) {
		return;
	} else {
		Postorder_Traverse_BinaryTree(node->left);
		Postorder_Traverse_BinaryTree(node->right);
		printf("%d ", node->idx);
	}
}

/* The deep for Binary-Tree */
static int BinaryTree_Deep(struct binary_node *node)
{
	int deep = 0;

	if (node != NULL) {
		int leftdeep  = BinaryTree_Deep(node->left);
		int rightdeep = BinaryTree_Deep(node->right);

		deep = leftdeep >= rightdeep ? leftdeep + 1: leftdeep + 1;
	}
	return deep;
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

	printf("Postorder Create BinaryTree\n");
	BiscuitOS_root = Postorder_Create_BinaryTree(BiscuitOS_root);

	/* Preoder traverse binary-tree */
	printf("Preorder Traverse Binary-Tree:\n");
	Preorder_Traverse_BinaryTree(BiscuitOS_root);
	printf("\n");

	/* Middorder traverse binary-tree */
	printf("Middorder Traverse Binary-Tree:\n");
	Middorder_Traverse_BinaryTree(BiscuitOS_root);
	printf("\n");

	/* Postorder traverse binary-tree */
	printf("Postorder Traverse Binary-Tree:\n");
	Postorder_Traverse_BinaryTree(BiscuitOS_root);
	printf("\n");

	/* The deep for Binary-Tree */
	printf("The Binary-Tree Deep: %d\n",
				BinaryTree_Deep(BiscuitOS_root));

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
binary-tree/Create/Postorder$ ./binary
Postorder Create BinaryTree
Preorder Traverse Binary-Tree:
200 143 754 6 3 386 7 9 876 486 8 2 740 1 5
Middorder Traverse Binary-Tree:
6 754 3 143 7 386 9 200 8 486 2 876 1 740 5
Postorder Traverse Binary-Tree:
6 3 754 7 9 386 143 8 2 486 1 5 740 876 200
The Binary-Tree Deep: 4
The Binary-Tree leaf: 8
{% endhighlight %}

--------------------------------------

#### <span id="运行分析">运行分析</span>

从运行的结果可以看出，在使用后序遍历二叉树的时候，二叉树遍历结果和预期一致

-----------------------------------------------

# <span id="附录">附录</span>

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
