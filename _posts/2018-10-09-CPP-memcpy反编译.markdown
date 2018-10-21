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

> 问题描述：定义一个字符串常量，使用两次内存拷贝函数后，反汇编查看地址字符串地址是否发生了变化。

### 反编译

- 1.编写main.cpp文件，在终端输入`g++ main.cpp -g -o out`，之所以在g++编译的时候加上-g是为了添加调试信息。<br/>

- 2.在终端输入`objdump -S -d ./out `，objdump中的-S选项是为了在显示汇编代码的时候同时显示原来的C++源代码。<br/>

![](http://owlypioka.bkt.clouddn.com/cpp_test2.png)


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
 18     memcpy(pStr,CONST_STR,strlen(CONST_STR));
 19     cout << pStr << endl;
 20     delete []pStr;                                                                                                
 21     return 0;
 22 }


```


部分反编译文件：<br/>

```
00000000000009ba <main>:
#include <cstring>

#define CONST_STR "dufaxing"

int main()
{
 9ba:	55                   	push   %rbp
 9bb:	48 89 e5             	mov    %rsp,%rbp
 9be:	48 83 ec 10          	sub    $0x10,%rsp
    char* pStr =  new char[20];
 9c2:	bf 14 00 00 00       	mov    $0x14,%edi
 9c7:	e8 64 fe ff ff       	callq  830 <_Znam@plt>
 9cc:	48 89 45 f8          	mov    %rax,-0x8(%rbp)
    memcpy(pStr,CONST_STR,strlen(CONST_STR));
 9d0:	48 8b 45 f8          	mov    -0x8(%rbp),%rax
 9d4:	ba 08 00 00 00       	mov    $0x8,%edx
 9d9:	48 8d 35 45 01 00 00 	lea    0x145(%rip),%rsi        # b25 <_ZStL19piecewise_construct+0x1>
 9e0:	48 89 c7             	mov    %rax,%rdi
 9e3:	e8 58 fe ff ff       	callq  840 <memcpy@plt>
    memcpy(pStr,CONST_STR,strlen(CONST_STR));
 9e8:	48 8b 45 f8          	mov    -0x8(%rbp),%rax
 9ec:	ba 08 00 00 00       	mov    $0x8,%edx
 9f1:	48 8d 35 2d 01 00 00 	lea    0x12d(%rip),%rsi        # b25 <_ZStL19piecewise_construct+0x1>
 9f8:	48 89 c7             	mov    %rax,%rdi
 9fb:	e8 40 fe ff ff       	callq  840 <memcpy@plt>
    cout << pStr << endl;
 a00:	48 8b 45 f8          	mov    -0x8(%rbp),%rax
 a04:	48 89 c6             	mov    %rax,%rsi
 a07:	48 8d 3d 12 06 20 00 	lea    0x200612(%rip),%rdi        # 201020 <_ZSt4cout@@GLIBCXX_3.4>
 a0e:	e8 4d fe ff ff       	callq  860 <_ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc@plt>
 a13:	48 89 c2             	mov    %rax,%rdx
 a16:	48 8b 05 b3 05 20 00 	mov    0x2005b3(%rip),%rax        # 200fd0 <_ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_@GLIBCXX_3.4>
 a1d:	48 89 c6             	mov    %rax,%rsi
 a20:	48 89 d7             	mov    %rdx,%rdi
 a23:	e8 48 fe ff ff       	callq  870 <_ZNSolsEPFRSoS_E@plt>
    delete []pStr;
 a28:	48 83 7d f8 00       	cmpq   $0x0,-0x8(%rbp)
 a2d:	74 0c                	je     a3b <main+0x81>
 a2f:	48 8b 45 f8          	mov    -0x8(%rbp),%rax
 a33:	48 89 c7             	mov    %rax,%rdi
 a36:	e8 45 fe ff ff       	callq  880 <_ZdaPv@plt>
    return 0;
 a3b:	b8 00 00 00 00       	mov    $0x0,%eax
}


```


- 由以上反汇编代码可以看出，两次调用memcpy函数常量字符串的地址没有变化。

### 打印常量字符串地址

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
 14 void str_test()
 15 {
 16     char* pStr =  new char[20];
 17     memcpy(pStr,CONST_STR,strlen(CONST_STR));
 18     memcpy(pStr,CONST_STR,strlen(CONST_STR));
 19     cout << "in str_test:" << &(CONST_STR) << endl;
 20     delete []pStr;
 21 }
 22 
 23 int main()
 24 {
 25     str_test();
 26     char* pStr =  new char[20];
 27     memcpy(pStr,CONST_STR,strlen(CONST_STR));
 28     cout << "in main:    " << &(CONST_STR) << endl;                                                                 
 29     delete []pStr;
 30     return 0;
 31 }                             

```

- 在终端中输入`g++ -o out main.cpp `执行生成的二进制文件`./out `

```
dufaxing@ubuntu:~/Projects/OOP/test_2$ vim main.cpp 
dufaxing@ubuntu:~/Projects/OOP/test_2$ g++ -o out main.cpp 
dufaxing@ubuntu:~/Projects/OOP/test_2$ ./out 
in str_test:0x55fdc3ff6bf5
in main:    0x55fdc3ff6bf5


```

### 总结

- 宏定义了字符串CONST_STR没有生成内存空间，在调用memcpy()函数后，CONST_STR在堆栈上创建了内存空间，并且在函数执行完成后CONST_STR的内存空间没有被销毁。所以在main()函数中两次调用memcpy()函数或者调用str_test()函数，CONST_STR的地址都没有变化。
