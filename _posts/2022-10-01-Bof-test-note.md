---
layout: post
title: Bof练习
subtitle: 常见的Bof内存溢出错误和应用
categories: markdown
tags: [Internet Security]
---



## 1.Bof题目

追究漏洞和入侵的缘由，超过一半的锅需要BOF（buffer over flow）来背。所以第一时间了解bof很有必要。下面这个例子通过一个故意构造的场景，演示了bof的一种典型情况。

下面的这个c程序，看似人畜无害，实则蕴含一个危险的漏洞，或者说暗藏后门。

```c
#include ...
main()
{
	char passwd[8] = {"??????"}; // 此处实际密码为2e4rfe
	char yourpasswd[8] = {""};
again:
	puts("please input passwd?");
	gets(yourpasswd);	
	if (strcmp(yourpasswd, passwd)= =0)
		goto ok;
	puts("passwd error");
	goto again;
	exit(-2);
ok:
	puts("correct!");
	// do work you want
	return 0;
}
```

你能运行对应的exe，争取执行到输出“correct!”吗？

## 2.题目解析

```c
gets(yourpasswd);	
```

本质上来说这一句代码就是真正的“罪魁祸首”。这段程序中蕴含了一个不安全的函数gets（char *p),gets（）会将用户输入的所有字符读取并且存放在以p为首地址的一个字符串中去。但存放的过程却会忽略p的数组容量。如果写入的内容超出了缓冲区的大小，超出的那一部分就会被写入到缓冲区的后方的地址中去，并且会把原有的数据全部覆盖掉。

如果我们不知道具体的密码，但是要尝试得到最后的”correct！''的结果，那么就有理由猜想，原有的存放password的数组passwd以及存放我们输入的密码的数组yourpasswd的地址距离，应该不会太远，甚至是临近的。对于C语言的内存分配机制，我们可以知晓：**C语言在为变量分配对应的内存地址的时候。总是从高地址位到低地址位分配的。**对应字符串的存储，我们可以猜测，passwd的地址应该比yourpasswd的地址高8个字节。那么也就是说，在我们输入yourpasswd的时候，我们可以输入多于8个字符来填充yourpasswd，并覆盖掉passwd中已有的内容。

不过需要注意的是，gets（）在读取完用户输入的字符串之后，会在最后再添加一个“/0"以示结束。那么我们能够输入的字符串长度应该为8-15个字符。少于这个长度就无法覆盖到passwd数组，超过这个长度就会重新构造一个全新的passwd数组。

因此我们可以做到：

```c
please input passwd?
1234567890
passwd error
please input passwd?
90
correct!
```

本质上90覆盖了新的passwd，因此再次输入就可以获得正确的提示了。

或许还会有更简单的方式——直接输入八位的密码，然后在判定错误后直接回车：

```c
please input passwd?
11111111
passwd error
please input passwd?
// 回车
correct!
```

