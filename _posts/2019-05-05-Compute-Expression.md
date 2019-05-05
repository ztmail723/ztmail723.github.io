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
求算数表达式的值。算数表达式按中缀给出，以=号结束，包括+,-,*,/四种运算和(、)分隔符。运算数的范围是非负整数，小于等于$10^9$。

C++代码
-------------------------------------

{% highlight c++%}
#include<iostream>
#include<stack>
#define SIZE 1005
using namespace std;
bool Isnum(char numword);//判断字符是否为数字
int Prior(char signword);//求优先级函数
unsigned int Thenum(char *Expression,int& i);//输出数字
unsigned int ComputeOne(unsigned int A,unsigned int B,char signword);//计算单个结果
unsigned int ComputeExpression(char *Expression);//输出表达式的结果
int main(void)
{
    char Expression[SIZE];//表达式数组
    cin>>Expression;//输入
    cout<<ComputeExpression(Expression);//输出计算结果
}
bool Isnum(char numword)
{
    return numword>='0'&&numword<='9'?true:false;
}
int Prior(char signword)
{
    switch(signword)
    {
    case ')':return 0;
    case '+':
    case '-':return 1;
    case '*':
    case '/':return 2;
    default :return 3;
    }
}
unsigned int Thenum(char *Expression,int& i)
{
    unsigned int tempnum=0;//临时数字
    while(Isnum(Expression[i]))
    {
        tempnum*=10;
        tempnum+=Expression[i++]-'0';
    }
    i--;//把指针i减回去
    return tempnum;
}
unsigned int ComputeOne(unsigned int A,unsigned int B,char signword)
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
unsigned int ComputeExpression(char *Expression)
{
    stack<unsigned int> NumStack;//数字栈
    stack<char> SignStack;//符号栈
    unsigned int tempB;//临时变量
    for(int i=0;Expression[i]!='=';i++)
    {
        if(Isnum(Expression[i]))
        {
            NumStack.push(Thenum(Expression,i));
        }
        else
        {
            while(!SignStack.empty()&&Prior(SignStack.top())>=Prior(Expression[i]))
            {
                if(SignStack.top()=='(')//如果遇到左括号，弹出并终止
                {
                    SignStack.pop();
                    break;
                }
                tempB=NumStack.top();
                NumStack.pop();
                NumStack.top()=ComputeOne(NumStack.top(),tempB,SignStack.top());
                SignStack.pop();
            }
            if(Expression[i]!=')')SignStack.push(Expression[i]);
        }
    }
    while(!SignStack.empty())
    {
        unsigned int tempB;
        tempB=NumStack.top();
        NumStack.pop();
        NumStack.top()=ComputeOne(NumStack.top(),tempB,SignStack.top());
        SignStack.pop();
    }
    return NumStack.top();
}
{% endhighlight%}
