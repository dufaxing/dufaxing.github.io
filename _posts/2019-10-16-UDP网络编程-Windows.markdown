---
layout: post
title:  "UDP网络编程（Windows）"
date:   2019-10-16 00:00:00
categories: 计算机基础
tags: C++ QT
excerpt: 在Windows下UDP网络编程
mathjax: true
---
* content
{:toc}
---


> [博客地址](https://dufaxing.com){:target="_blank"}



### 基于qt的网络编程

在QT中编写网络发报端/客户端，时build时出现<br/>
error: undefined reference to `_imp__WSAStartup@8’等，很多网络类似的错误<br/>
等大约10条error，原因是socket库的编译链接问题。<br/>


错误原因：因为没有联接socket库ws2_32.lib。因此要链接该库<br/>
总结：所有运用到WinSock2的程序在编译连接时都要用的该库<br/>

解决办法：<br/>
在项目的pro文件中添加：<br/>
`LIBS += -lpthread libwsock32 libws2_32`

---

### Windows下UDPServer


```
#include<winsock2.h>
#include<stdio.h>
#include<string.h>
#include<iostream>
using namespace std;
#pragma comment(lib,"ws2_32.lib")
#define BUFFER_SIZE 1024
int main()
{
    WSADATA WSAData;
    char receBuf[BUFFER_SIZE];
    char Response[] = "";
    if (WSAStartup(MAKEWORD(2, 2), &WSAData) != 0) {
        printf("初始化失败");
        exit(1);

    }
    SOCKET sockServer = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP);
    if (sockServer == INVALID_SOCKET) {
        printf("Failed socket() \n");
        return 0;

    }
    SOCKADDR_IN addr_Server; //服务器的地址等信息
    addr_Server.sin_family = AF_INET;
    addr_Server.sin_port = htons(4567);//端口号设置为4567
    addr_Server.sin_addr.S_un.S_addr = INADDR_ANY;
    if (bind(sockServer, (SOCKADDR*)&addr_Server, sizeof(addr_Server)) == SOCKET_ERROR) {
//服务器与本地地址绑定
        printf("Failed socket() %d \n", WSAGetLastError());
        return 0;

    }
    SOCKADDR_IN addr_Clt;

    int fromlen = sizeof(SOCKADDR);
    while (true) {
        int last = recvfrom(sockServer, receBuf, 1024, 0, (SOCKADDR*)&addr_Clt, &fromlen);
        if (last>0) {
            //判断接收到的数据是否为空
            receBuf[last] = '\0';//给字符数组加一个'\0'，表示结束了。不然输出有乱码
            if (strcmp(receBuf, "bye") == 0) {
                cout << " 客户端关闭..." << endl;
                closesocket(sockServer);
                return 0;

            } else {
                printf("接收到数据（%s）：%s\n", inet_ntoa(addr_Clt.sin_addr), receBuf);

            }

        }
        cout << "回复客户端消息:";
        cin >> Response; //给客户端回复消息
        sendto(sockServer, Response, strlen(Response), 0, (SOCKADDR*)&addr_Clt, sizeof(SOCKADDR));

    }

    closesocket(sockServer);

    WSACleanup();
    return 0;
}

```


---

### Windows下UDPClient


```
#include<winsock2.h>
#include<stdio.h>
#include<string.h>
#include<iostream>
using namespace std;
#pragma comment(lib,"ws2_32.lib")
# define BUFFER_SIZE 1024 //缓冲区大小
int main()
{
    SOCKET sock_Client; //客户端用于通信的Socket
    WSADATA WSAData;
    char receBuf[BUFFER_SIZE]; //发送数据的缓冲区
    char sendBuf[BUFFER_SIZE]; //接受数据的缓冲区

    if (WSAStartup(MAKEWORD(2, 2), &WSAData) != 0) {
        printf("初始化失败!");
        return -1;
    }

    //初始化
    sock_Client = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP);//创建客户端用于通信的Socket
    SOCKADDR_IN addr_server; //服务器的地址数据结构
    addr_server.sin_family = AF_INET;
    addr_server.sin_port = htons(4567);//端口号为4567
    addr_server.sin_addr.S_un.S_addr = inet_addr("127.0.0.1"); //127.0.0.1为本电脑IP地址
    SOCKADDR_IN sock;
    int len = sizeof(sock);
    while (true) {
        cout << "请输入要传送的数据:";
        cin >> sendBuf;
        sendto(sock_Client, sendBuf, strlen(sendBuf), 0, (SOCKADDR*)&addr_server, sizeof(SOCKADDR));
        //int last=recv(sock_Client, receBuf, strlen(receBuf), 0); // (调用recv和recvfrom都可以)
        int last = recvfrom(sock_Client, receBuf, strlen(receBuf), 0, (SOCKADDR*)&sock, &len);
        if (last>0) {
            receBuf[last] = '\0'; //给字符数组加一个'\0'，表示结束了。不然输出有乱码
            if (strcmp(receBuf, "bye") == 0) {
                cout << "服务器关闭" << endl;//当服务器发来bye时，关闭socket
                closesocket(sock_Client);
                break;
            } else {
                printf("接收到数据：%s\n", receBuf);
            }

        }

    }
    closesocket(sock_Client);
    WSACleanup();


    return 0;
}
```


---


