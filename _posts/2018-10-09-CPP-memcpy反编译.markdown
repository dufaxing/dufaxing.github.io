---
layout: post
title:  "Memcpy反编译字符串常量"
date:   2018-10-09 00:00:00
categories: C++
tags: Linux
excerpt: 反编译
mathjax: true
---
* content
{:toc}

> 问题描述：定义一个字符串常量，使用内存拷贝函数后，反汇编查看地址



- 1.编写main.cpp文件，在终端输入`g++ main.cpp -g -o out`，之所以在g++编译的时候加上-g是为了添加调试信息。<br/>

- 2.在终端输入`objdump -S -d ./out `，objdump中的-S选项是为了在显示汇编代码的时候同时显示原来的C语言源代码。<br/>

![](http://owlypioka.bkt.clouddn.com/cpp_test_2.png)


源码如下：<br/>

```
  1 /*************************************************************************
  2     > File Name: main.cpp
  3     > Author: dufaxing
  4     > Mail: dufaxing@qq.com
  5     > Created Time: Tue 09 Oct 2018 02:18:06 AM PDT
  6  ************************************************************************/
  7 
  8 #include<iostream>
  9 using namespace std;
 10 #include <cstring>
 11 
 12 #define CONST_STR "dufaxing"
 13                                                                                                                   
 14 int main()
 15 {
 16     char* pStr =  new char[20];
 17     memcpy(pStr,CONST_STR,strlen(CONST_STR));
 18     cout << pStr << endl;
 19     return 0;
 20 }

```


部分反编译文件：<br/>

```
000000000000096a <main>:
#include <cstring>

#define CONST_STR "dufaxing"

int main()
{
 96a:	55                   	push   %rbp
 96b:	48 89 e5             	mov    %rsp,%rbp
 96e:	48 83 ec 10          	sub    $0x10,%rsp
    char* pStr =  new char[20];
 972:	bf 14 00 00 00       	mov    $0x14,%edi
 977:	e8 74 fe ff ff       	callq  7f0 <_Znam@plt>
 97c:	48 89 45 f8          	mov    %rax,-0x8(%rbp)
    memcpy(pStr,CONST_STR,strlen(CONST_STR));
 980:	48 8b 45 f8          	mov    -0x8(%rbp),%rax
 984:	ba 08 00 00 00       	mov    $0x8,%edx
 989:	48 8d 35 25 01 00 00 	lea    0x125(%rip),%rsi        # ab5 <_ZStL19piecewise_construct+0x1>
 990:	48 89 c7             	mov    %rax,%rdi
 993:	e8 68 fe ff ff       	callq  800 <memcpy@plt>
    cout << pStr << endl;
 998:	48 8b 45 f8          	mov    -0x8(%rbp),%rax
 99c:	48 89 c6             	mov    %rax,%rsi
 99f:	48 8d 3d 7a 06 20 00 	lea    0x20067a(%rip),%rdi        # 201020 <_ZSt4cout@@GLIBCXX_3.4>
 9a6:	e8 75 fe ff ff       	callq  820 <_ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc@plt>
 9ab:	48 89 c2             	mov    %rax,%rdx
 9ae:	48 8b 05 1b 06 20 00 	mov    0x20061b(%rip),%rax        # 200fd0 <_ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_@GLIBCXX_3.4>
 9b5:	48 89 c6             	mov    %rax,%rsi
 9b8:	48 89 d7             	mov    %rdx,%rdi
 9bb:	e8 70 fe ff ff       	callq  830 <_ZNSolsEPFRSoS_E@plt>
    return 0;
 9c0:	b8 00 00 00 00       	mov    $0x0,%eax
}


}

```