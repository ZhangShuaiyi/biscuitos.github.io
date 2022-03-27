---
layout: post
title:  "后序遍历二叉树"
date:   2019-05-12 05:30:30 +0800
categories: [HW]
excerpt: TREE 后序遍历二叉树.
tags:
  - Tree
---

![DTS](/assets/PDB/BiscuitOS/kernel/IND00000T.jpg)

> [Github: 后序遍历二叉树](https://github.com/BiscuitOS/HardStack/tree/master/Algorithem/tree/binary-tree/IterateOver/Postorder)
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

在二叉树中，后序遍历二叉树指的是每当遍历到一个节点，那么先遍历左孩子，然后遍历右孩子，
最后遍历当前节点。因此，可以使用递归实现这一逻辑，如下：

{% highlight ruby %}
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
{% endhighlight %}

在上面的实现中，如果遍历到的节点为 NULL 那么直接返回。如果不为 NULL，那么就先遍历当
前节点的左孩子，遍历完左孩子，再遍历当前节点的右孩子，最后处理当前节点，此处输出当前
节点的 idx 变量。以此遍历下去，整棵树上的所有节点都被遍历到。接下来通过
实践进一步认知后序遍历二叉树。实践之前准备了一组二叉树后序排列的数据，如下：

{% highlight ruby %}
static int Perfect_BinaryTree_data[] = {
                                  200, 143, 754, 6, -1, -1, 3, -1, -1,
                                  386, 7, -1, -1, 9, -1, -1, 876, 486,
                                  8, -1, -1, 2, -1, -1, 740, 1, -1, -1,
                                  5, -1, -1 };
{% endhighlight %}

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

> [实践源码 binary.c on GitHub](https://github.com/BiscuitOS/HardStack/blob/master/Algorithem/tree/binary-tree/IterateOver/Postorder/binary.c)

开发者也可以使用如下命令获得：

{% highlight ruby %}
wget https://raw.githubusercontent.com/BiscuitOS/HardStack/master/Algorithem/tree/binary-tree/IterateOver/Postorder/binary.c
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

	/* Postorder traverse binary-tree */
	printf("Postorder Traverse Binary-Tree:\n");
	Postorder_Traverse_BinaryTree(BiscuitOS_root);
	printf("\n");

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
binary-tree/IterateOver/Postorder$ ./binary
Preorder Create BinaryTree
Postorder Traverse Binary-Tree:
6 3 754 7 9 386 143 8 2 486 1 5 740 876 200
{% endhighlight %}

--------------------------------------

#### <span id="运行分析">运行分析</span>

从运行的结果可以看出，在使用后序遍历二叉树的时候，二叉树遍历结果和预期一致

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
