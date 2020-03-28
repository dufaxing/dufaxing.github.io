---
layout: post
title:  "unordered_map哈希表的应用"
date:   2020-03-26 00:00:00
categories: 计算机基础
tags: 算法
excerpt: LeetCode第49题： 字母异位词分组
mathjax: true
---
* content
{:toc}
---

![KYHHbj.png](https://s2.ax1x.com/2019/10/23/KYHHbj.png)



> [博客地址](https://dufaxing.com){:target="_blank"}


### 题目描述

>给定一个字符串数组，将字母异位词组合在一起。字母异位词指字母相同，但排列不同的字符串。<br>

示例:
```
输入: ["eat", "tea", "tan", "ate", "nat", "bat"],
输出:
[
  ["ate","eat","tea"],
  ["nat","tan"],
  ["bat"]
]
```
说明：

所有输入均为小写字母。
不考虑答案输出的顺序。


---

### c++ map与unordered_map区别及使用

#### 需要引入的头文件不同

`map: #include < map >`
`unordered_map: #include < unordered_map >`

#### 内部实现机理不同
- map： 
  - map内部实现了一个红黑树（红黑树是非严格平衡二叉搜索树，而AVL是严格平衡二叉搜索树），红黑树具有自动排序的功能，因此map内部的所有元素都是有序的，红黑树的每一个节点都代表着map的一个元素。因此，对于map进行的查找，删除，添加等一系列的操作都相当于是对红黑树进行的操作。
  - map中的元素是按照二叉搜索树（又名二叉查找树、二叉排序树，特点就是左子树上所有节点的键值都小于根节点的键值，右子树所有节点的键值都大于根节点的键值）存储的，使用中序遍历可将键值按照从小到大遍历出来。

- unordered_map: 
  - unordered_map内部实现了一个哈希表（也叫散列表，通过把关键码值映射到Hash表中一个位置来访问记录，查找的时间复杂度可达到O(1)，其在海量数据处理中有着广泛应用）。因此，其元素的排列顺序是无序的。


#### 优缺点以及适用处

map：

- 优点：有序性，这是map结构最大的优点，其元素的有序性在很多应用中都会简化很多的操作
红黑树，内部实现一个红黑书使得map的很多操作在lgn的时间复杂度下就可以实现，因此效率非常的高
- 缺点： 空间占用率高，因为map内部实现了红黑树，虽然提高了运行效率，但是因为每一个节点都需要额外保存父节点、孩子节点和红/黑性质，使得每一个节点都占用大量的空间

- 适用处：对于那些有顺序要求的问题，用map会更高效一些

 

unordered_map：

- 优点： 因为内部实现了哈希表，因此其查找速度非常的快
- 缺点： 哈希表的建立比较耗费时间
- 适用处：对于查找问题，unordered_map会更加高效一些，因此遇到查找问题，常会考虑一下用unordered_map

总结：

内存占有率的问题就转化成红黑树 VS hash表 , 还是unorder_map占用的内存要高。
但是unordered_map执行效率要比map高很多。<br>
对于unordered_map或unordered_set容器，其遍历顺序与创建该容器时输入的顺序不一定相同，因为遍历是按照哈希表从前往后依次遍历的。<br>
map和unordered_map的使用：<br>
unordered_map的用法和map是一样的，提供了 insert，size，count等操作，并且里面的元素也是以pair类型来存贮的。其底层实现是完全不同的，上方已经解释了，但是就外部使用来说却是一致的。


#### find函数

`iterator find ( const key_type& key );`<br>
如果key存在，则find返回key对应的迭代器，如果key不存在，则find返回unordered_map::end。因此可以通过`map.find(key) == map.end()`来判断，key是否存在于当前的unordered_map中。


```
// unordered_map::find
#include <iostream>
#include <string>
#include <unordered_map>

int main ()
{
  std::unordered_map<std::string,double> mymap = {
     {"mom",5.4},
     {"dad",6.1},
     {"bro",5.9} };

  std::string input;
  std::cout << "who? ";
  getline (std::cin,input);

  std::unordered_map<std::string,double>::const_iterator got = mymap.find (input);

  if ( got == mymap.end() )
    std::cout << "not found";
  else
    std::cout << got->first << " is " << got->second;

  std::cout << std::endl;

  return 0;
}
```

#### count函数

`size_type count ( const key_type& key ) const`

count函数用以统计key值在unordered_map中出现的次数。实际上，c++ unordered_map不允许有重复的key。因此，如果key存在，则count返回1，如果不存在，则count返回0.

```
// unordered_map::count
#include <iostream>
#include <string>
#include <unordered_map>

int main ()
{
  std::unordered_map<std::string,double> mymap = {
     {"Burger",2.99},
     {"Fries",1.99},
     {"Soda",1.50} };

  for (auto& x: {"Burger","Pizza","Salad","Soda"}) {
    if (mymap.count(x)>0)
      std::cout << "mymap has " << x << std::endl;
    else
      std::cout << "mymap has no " << x << std::endl;
  }

  return 0;
}
```

---

### 代码

```
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        vector<vector<string>> res;
        unordered_map<string,int> work;
        int sub = 0;

        for(string str:strs) {
            string temp = str;
            sort(temp.begin(),temp.end());//将字母异位词排序
            if (work.count(temp)) {//查找哈希表中是否存在过该 字母异位词排序
                res[work[temp]].push_back(str);
            } else {
                vector<string> vec(1,str);
                res.push_back(vec);
                work[temp] = sub++;//保存该 字母异位词排序 的数组下标
            }
        }
        return res;
    }
};
```

---
