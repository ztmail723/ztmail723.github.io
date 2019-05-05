---
layout: post
title:  判断给定二叉树是否为完全二叉树(C++)
date:   2019-05-04 15:00:00 +0800
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


什么是完全二叉树
-------------------------------------
一棵非空高度为$k(k>=0)$的满二叉树，是有$2^{k-1}-1$个结点的二叉树。   
一棵包含$n$个结点高度为$k$的二叉树$T$，当按层次顺序编号$T$的所有结点，对应于一棵高度为$k$的满二叉树中编号由$1$至$n$的那些结点时，$T$被称为完全二叉树(complete binary)。  

C++代码
-------------------------------------

{% highlight c++%}
#include<iostream>
#include<queue>
using namespace std;
struct Tree//树节点的结构体定义
{
    int value;
    Tree* left;
    Tree* right;
};
bool judge(Tree *root)//传入树的根节点，判断该树是否为完全二叉树
{
    bool flag=false;//结束标记默认未打开
    queue<Tree*> q;//临时队列
    q.push(root);//根节点入队
    while(!q.empty())//队列不为空时循环不停止
    {
        if(p==NULL)//如果当前节点为空
        {
            flag=true;//结束标记打开
        }
        else if(flag==true)//如果不为空，且结束标记已打开
        {
            return false;//此时返回假
        }
        else//不为空，且结束标记未打开
        {
            q.push(q.front->left);
            q.push(q.front->right);//让左右孩子入队
        }
        q.pop;//队首节点出队
    }
    return true;//搜索完毕，返回真
}
{% endhighlight%}
