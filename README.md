﻿	Author: Zoo
	QQ:276793422

		模块说明
	当前模块是一个支持跨平台的全动态申请的前缀树，
	由于前一阵子需要用到一个前缀树，
	本来想在网上找一个，但是我就没有找到全动态申请内存的前缀树，
	网上的各种前缀树都是子列表元素个数确定的，
	但是我用前缀树的目的是用来存放文件路径，
	文件路径的字符串是包含中文字符的，
	这样，如果把子列表元素个数确定的话，
	那么就可能需要每个树节点都需要0xFFFF个指针元素，
	如果真的是这样，就疯了，内存浪费过多，x86环境，一个节点就要用掉256K内存，x64下翻倍，
	所以我的需求是找到一个全动态内存申请的前缀树，
	可是，我找了各大平台，各大开源代码，都没有找到，
	也可能是我个人水平不足，寻找的不够多，
	所以我自己写了一个全动态内存申请的前缀树，
	使用起来相对比较麻烦，但是抽象出了几乎全部需要平台支持的部分，
	使用时，需要在各自平台上支持一系列的内存申请释放相关函数，
	然后就可以随意用了，
	所有接口全部封装在内部，外部不需要知道任何结构体信息。

		当前模块缺陷
	1：从右侧开始寻找规则（已修复，可以支持从右侧开始找规则）
		模糊查找时，可以从根节点开始寻找信息，
		这样本身是没问题的，但是实际上不是很合逻辑，
		因为道理上来说需要如果对一个文件做操作，它所在的目录才应该影响它的行为，而不是根目录。
		比如，列表中有四条规则 
			D:\\1\\2\\3\\4.c
			D:\\1\\2\\3\\
			D:\\1\\2\\
			D:\\1\\
		这时如果我要查询 D:\\1\\2\\3\\4.c 这个路径的规则
			那么，精确查找的话，会直接找到 D:\\1\\2\\3\\4.c 这条路径的规则，
		但是如果我要查询 D:\\1\\2\\3\\5.c 这个路径的规则
			由于实际上对应路径的规则不存在，那么精确查找就会失败
			但是如果用现有的模糊查找，只能先找到 D:\\1\\ 目录的规则
				但是实际上 D:\\1\\2\\3\\ 这个目录的规则才是应该找到的
		以上这个缺陷修复之后，实际上就可以支持真正的删除节点了，
			做法就是，先寻找要删除的节点，如果找到了的话，就循环向上层寻找节点，
			如果找到的节点有同级节点，那么就删除当前节点，然后返回，
			如果找到的节点没有同级节点，那么删除当前节点之后，还可以继续向上找，

	2：内存使用过于繁琐
		因为每个节点在寻找它的同级节点的时候，使用的是next 成员，
		这么做有好处，就是这样做可以保证整个前缀树的所有节点都长成一个样子，
			这样做的话，我们就可以用那些按照指定长度来申请内存的池来申请内存了，比如Windows 的Lookaside，
		但是缺点就是，访问链表的速度，远低于访问数组的速度，
			而我的树，很可能就是个读取密集型的树。
		所以这里怎么改，还是个问题，如果用了数组索引型的话，那么内存的问题肯定不可避免了。
		现有计划是
			1）单独拆分个工程来做支持数组型的节点索引模式
			2）用一个开关，让这个树支持两种节点索引模式

	目前问题是测试用例不足，写了大部分用例，但是有些仍然不够