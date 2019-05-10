---
layout: post
title:  表达式计算(C++)
date:   2019-05-05 08:00:00 +0800
categories: 技术文档
tag: 算法
---
<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>


* content
{:toc}


问题描述
-------------------------------------
求算数表达式的值。算数表达式按中缀给出，以=号结束，包括+,-,\*,/四种运算和(、)分隔符。运算数的范围是非负整数，小于等于$10^9$。

思路解析
-------------------------------------

请看以下两个例子：

    表达式1：1*2+3=
    表达式2：1+2*3=

对于表达式1，我们是从左到右进行的运算，我们的心算步骤大概如下：

    计算1*2+3
    步骤1：1*2=2
    步骤2：2+3=5
    最终结果：5

用计算机的思维来描述一下表达式1的计算吧！

    对于字符串"1*2+3"
    读取字符串的第1个字符'1'，增加元素1
    读取字符串的第2个字符'*'，增加元素*
    读取字符串的第3个字符'2'，增加元素2
    读取字符串的第4个字符'+'，增加元素+
    读取字符串的第5个字符'3'，增加元素3
    目前存储的元素：1 * 2 + 3
    计算1*2=2，删除元素'1'、'*'和'2'，增加元素'2'
    目前存储的元素：2 + 3
    计算2+3=5，删除元素'2'、'+'和'3'，增加元素'5'
    目前存储的元素：5
    只剩一个元素，即为最终的结果
    所以1*2+3=5

对于表达式2，我们是先算乘法再算加法，我们的心算步骤大概如下：

    计算1+2*3
    步骤1：2*3=6
    步骤2：1+6=7
    最终结果：7

尝试用计算机的思维来描述一下表达式2的计算：

    对于字符串"1+2*3"
    读取字符串的第1个字符'1'，增加元素1
    读取字符串的第2个字符'+'，增加元素+
    读取字符串的第3个字符'2'，增加元素2
    读取字符串的第4个字符'*'，增加元素*
    读取字符串的第5个字符'3'，增加元素3
    目前存储的元素：1 + 2 * 3
    计算2*3=6，删除元素'2'、'*'和'3'，增加元素'6'
    目前存储的元素：1 + 6
    计算1+6=7，删除元素'1'、'+'和'6'，增加元素'7'
    目前存储的元素：7
    只剩一个元素，即为最终的结果
    所以1+2*3=7

发现了吗？对于表达式2，我们的计算顺序是“先读入的元素后计算，后读入的元素先计算”，是不是想到了栈【先进后出，后进先出】的特点？  
但倘若是这样【从后向前算】，又如何计算表达式1呢？

    对于字符串"1*2+3"
    读取字符串的第1个字符'1'，增加元素1
    读取字符串的第2个字符'*'，增加元素*
    读取字符串的第3个字符'2'，增加元素2
    【计算1*2，删除'1'、'*'、'2'，增加2】
    【当前元素：2】
    读取字符串的第4个字符'+'，增加元素+
    读取字符串的第5个字符'3'，增加元素3
    
没错，我们是【从后向前算】，但对于字符串的读取是【从前向后读】。  
那么，在读取的过程中，让表达式【顺序计算】的部分先算完，【从后向前计算】的部分存起来。  
比如如下表达式：  

    1+2+3*4+5*6=

我们先把1+2=3算完，这样在读取到3\*4的时候表达式变成了：

    3+3*4

在读取到+时，将【3+3\*4】看作一个整体A，加号后边的元素看做一个整体B，A+B是【顺序计算】的部分（加号不会比A中的乘号先算）。  
也就是说，我们虽然不确定B内部的具体计算顺序，但至少A这个整体相对于B先进行计算。  
而我们已经确保了A是一个【从后向前计算】的序列，那么我们就可以用【从后向前计算】的方式计算出A的值为15。  
所以后来的表达式为

    15+B
    
然后我们接着读取字符串，将B细化

    15+5*6
    
读到等于号'='结束，发现最终留下的又是一个【从后向前计算】的序列，我们用【从后向前计算】的方式最终得出表达式的结果为45。  
也就是说，整体思路是在读取过程中，随时解决掉【顺序计算】的部分，从而始终保持存储的元素序列是一个可以用栈模拟的【从后向前计算】的模型。  

具体实现思路
-------------------------------------

建立两个栈，一个为NumStack为数字栈，一个叫SignStack为符号栈。  
对每种符号设定优先级，整个读取过程中保持【符号的优先级递增】。  

C++代码
-------------------------------------

{% highlight c++%}
#include<iostream>
#include<stack>
#define SIZE 1005
using namespace std;
bool Isnum(char numword);//判断字符是否为数字
int Prior(char signword);//求优先级函数
unsigned long long Thenum(char *Expression,int& i);//输出数字
unsigned long long ComputeOne(unsigned long long A,unsigned long long B,char signword);//计算单个结果
unsigned long long ComputeExpression(char *Expression);//输出表达式的结果
int main(void)
{
    char Expression[SIZE];//表达式字符数组
    cin>>Expression;//输入
    cout<<ComputeExpression(Expression);//输出计算结果
    return 0;
}
bool Isnum(char numword)
{
    return numword>='0'&&numword<='9'?true:false;
}
int Prior(char signword)
{
    switch(signword)
    {
    case '=':return -1;//等于号作为哨兵项
    case '(':
    case ')':return 0;
    case '+':
    case '-':return 1;
    case '*':
    case '/':return 2;
    default: return 3;
    }
}
unsigned long long Thenum(char *Expression,int& i)
{
    unsigned long long tempnum=0;//临时数字
    while(Isnum(Expression[i]))
    {
        tempnum*=10;
        tempnum+=Expression[i++]-'0';
    }
    i--;//把引用的i减回去
    return tempnum;
}
unsigned long long ComputeOne(unsigned long long A,unsigned long long B,char signword)
{
    switch(signword)
    {
        case '+':return A+B;
        case '-':return A-B;
        case '*':return A*B;
        case '/':return A/B;
        default:return 0;
    }
}
unsigned long long ComputeExpression(char *Expression)
{
    stack<unsigned long long> NumStack;//数字栈
    stack<char> SignStack;//符号栈
    for(int i=0;Expression[i]!='\0';i++)
    {
        if(Isnum(Expression[i])) NumStack.push(Thenum(Expression,i));
        else
        {
            for(unsigned long long tempB;\
                !SignStack.empty()&&Prior(SignStack.top())>=Prior(Expression[i])&&Expression[i]!='(';\
                SignStack.pop())
            {
                if(SignStack.top()=='(')//遇到左括号，强制弹出
                {
                    SignStack.pop();
                    break;
                }
                tempB=NumStack.top();
                NumStack.pop();
                NumStack.top()=ComputeOne(NumStack.top(),tempB,SignStack.top());
            }
            if(Expression[i]!=')')SignStack.push(Expression[i]);
        }
    }
    return NumStack.top();
}

{% endhighlight%}

输入样例

    (5+(2+3)*4-3)=

输出样例

    22

思考题
-------------------------------------
1. 将中缀表达式转换为后缀表达式。
2. 将后缀表达式转换为中缀表达式。（关于后缀表达式，见拓展）
3. 增加一个运算符"^"代表乘方，优先级高于乘除，小于括号。例如 2\*(1+2\*3)^4 代表 $$ 2*{(1+2*3)}^4 $$ 的结果。

拓展
-------------------------------------
针对后缀表达式（逆波兰式）的计算，用一个栈很容易实现。例如：

    5 1 2 + 4 * + 3 -

这样的后缀表达式与如下中缀表达式等效

    5+((1+2)*4)-3

用栈模拟后缀表达式的计算结果如下

    状态：5        (检测到5，5入栈)   
    状态：5 1      (检测到1，1入栈)  
    状态：5 1 2    (检测到2，2入栈)  
    状态：5 3      (检测到+，2出栈，1出栈，1+2入栈)  
    状态：5 3 4    (检测到4，4入栈)  
    状态：5 12     (检测到*，4出栈，3出栈，3*4入栈)  
    状态：17       (检测到+，12出栈，5出栈，5+12入栈)  
    状态：17 3     (检测到3，3入栈)  
    状态：14       (检测到-，3出栈，17出栈，17-3入栈)  

可以看出，针对后缀表达式的计算只需要使用一个栈，用于存储数字。而对于中缀表达式，并不是那么容易。  
如果想要计算5+((1+2)\*4)-3这样的中缀表达式，难点有二：  
1. 中缀表达式的数字出现在运算符的两边，不容易存储。  
2. 从左到右扫描到的运算符并不是【立即参与运算】，而是具有一定的优先级。  
