---
typora-root-url: D:\code\git\leetcode\img
---

# kamar替换数字

> [题目链接](https://kamacoder.com/problempage.php?pid=1064)
>
> [文章链接](https://programmercarl.com/kama54.%E6%9B%BF%E6%8D%A2%E6%95%B0%E5%AD%97.html#%E6%80%9D%E8%B7%AF)

给定一个字符串 s，它包含小写字母和数字字符，请编写一个函数，将字符串中的字母字符保持不变，而将每个数字字符替换为number。

例如，对于输入字符串 "a1b2c3"，函数应该将其转换为 "anumberbnumbercnumber"。

对于输入字符串 "a5b"，函数应该将其转换为 "anumberb"

输入：一个字符串 s,s 仅包含小写字母和数字字符。

输出：打印一个新的字符串，其中每个数字字符都被替换为了number

样例输入：a1b2c3

样例输出：anumberbnumbercnumber

数据范围：1 <= s.length < 10000

**思路**

![](/替换数字.png)

从前向后填充就是O(n^2)的算法了，因为每次添加元素都要将添加元素之后的所有元素整体向后移动。

**其实很多数组填充类的问题，其做法都是先预先给数组扩容带填充后的大小，然后在从后向前进行操作。**

这么做有两个好处：

1. 不用申请新数组。
2. 从后向前填充元素，避免了从前向后填充元素时，每次添加元素都要将添加元素之后的所有元素向后移动的问题。

```c++
#include <iostream>
using namespace std;
int main()
{
    string s;
    cin>>s;
    
    int oldSize = s.size();
    int numCount = 0;
    for(char c:s)
    {
        if(c>='0'&&c<='9')
        {
            numCount++;
        }
    }
    s.resize(s.size()+numCount*5);;
    int newSize = s.size();
    for(int oldPtr = oldSize-1,newPtr = newSize-1;oldPtr>=0;oldPtr--)
    {
        if(s[oldPtr]>='a'&&s[oldPtr]<='z')
            s[newPtr--] = s[oldPtr];
        else
        {
            s[newPtr--] =  'r';
            s[newPtr--] =  'e';
            s[newPtr--] =  'b';
            s[newPtr--] =  'm';
            s[newPtr--] =  'u';
            s[newPtr--] =  'n';
        }
    }
    cout<<s<<endl;
}
```

