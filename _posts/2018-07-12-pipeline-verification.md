---
layout: post
title:  "cpu流水线验证"
categories: 系统结构
tags:  系统结构 c++
author: ganpeng
---

* content
{:toc}

上系统结构课的时候老师留了一个实验作业。题目是如何使用代码验证cpu中流水线机制的存在？


## 原理

cpu中流水线机制的存在是为了更快地执行指令。其将一条指令的执行过程分成许多个周期，例如mips五段流水线分成下面五个阶段：
- 取指
- 译码
- 执行
- 访存
- 写回

分阶段执行使得指令的平均时钟周期减少，但有一个问题就是跳转指令的存在。
跳转指令的结果只有在代码实际执行时才能知道，因此cpu中流水线会停顿几个时钟周期。而cpu为了能省下这几个时钟周期来更快地执行代码使用了分支预测机制。

分支预测就是在碰到跳转指令时对该指令结果是跳转还是不跳转进行预测，根据预测结果取跳转地址的指令进行译码。分支预测主要分为静态预测和动态预测，具体可参考维基百科:[分支预测器](https://zh.wikipedia.org/wiki/%E5%88%86%E6%94%AF%E9%A0%90%E6%B8%AC%E5%99%A8)。当分支预测器预测错误时需要将预取的指令清空，从正确的指令地址重新取指，由于现在的cpu流水线长度较长，因此预测错误的代价非常高，这将会使流水线的效率大大降低。

现在回到如何验证流水线存在的问题上，在知道了cpu中分支预测机制存在的情况下，可以写一个无法预测的代码来打乱cpu分支预测器的预测结果。使流水线效率降低，从而和没有流水线的cpu，或者说是单周期cpu区分开来。


## 代码

下面我们用代码来验证cpu中是否存在流水线。
我的做法是用一个数组，该数组中每个元素都是0~255的随机数，然后遍历这个数组，将数组中大于128的数相加，接着我们将数组排序，同样遍历数组并将大于128的数相加。
```c++
#include <algorithm>
#include <ctime>
#include <iostream>
int main()
{
	const int size = 10000;//数组大小
	int data[size];
	long long sum = 0;//数组中大于128的数的和
	double t;
	clock_t start;//计算程序运行时间用的
	std::srand(time(0));//初始化随机数种子
	for (int i = 0; i < size; ++i)
		data[i] = std::rand() % 256;//用0~255的数初始化数组
	start = clock();//开始计时
	for (int i = 0; i < size * 10; ++i)//由于遍历一遍数组时间太短，看不出来差别，因此重复多次遍历数组
	{
		for (int c = 0; c < size; ++c)
		{
			if (data[c] >= 128)
				sum += data[c];
		}
	}
	t = static_cast<double>(clock() - start) / CLOCKS_PER_SEC;//结束计时
	std::cout << "before sort: " << t << "s" << std::endl;//耗时
	std::cout << "sum= " << sum << std::endl;//和

	sum = 0;
	std::sort(data, data + size);//排序
	start = clock();
	for (int i = 0; i < size * 10; ++i)
	{
		for (int c = 0; c < size; ++c)
		{
			if (data[c] >= 128)
				sum += data[c];
		}
	}
	t = static_cast<double>(clock() - start) / CLOCKS_PER_SEC;
	std::cout << "after sort: " << t << "s" << std::endl;
	std::cout << "sum= " << sum << std::endl;
	return 0;
}
```

## 结论

代码运行结果如下

![result](/static/pipeline.png)


不管数组是否有序，cpu执行的代码数量都是一样的(求和结果也一样），这一点通过objdum反汇编也能看到两个for循环语句的汇编指令完全一样，因此如果cpu中不存在流水线的话每条指令执行时间应该都是一个时钟周期，因此时间应该一样，但是，由于cpu中确实存在流水线，因此数组排序前和排序后用时不一样。