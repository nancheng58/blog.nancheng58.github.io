---
title: Splay学习笔记
date: 2020-05-19 22:35:29
tags: 数据结构
categories: 学习笔记
---

# Splay学习笔记

高中时的笔记，重新发布一下。
图好像都丢了，网上一堆，可以搜一下。

[](#Splay "Splay")Splay

[](#类别：二叉排序树 "类别：二叉排序树")类别：二叉排序树
================================

空间复杂度：O(n)

时间复杂度：O(log n)内完成插入、查找、删除操作

创造者：Daniel Sleator和Robert Tarjan

优点：每次查询会调整树的结构，使被查询频率高的条目更靠近树根。

树的旋转是splay的基础，对于二叉查找树来说，树的旋转不破坏查找树的结构。

[](#Splaying "Splaying")Splaying
--------------------------------

Splaying是Splay Tree中的基本操作，为了让被查询的条目更接近树根，Splay Tree使用了树的旋转操作，同时保证二叉排序树的性质不变。

Splaying的操作受以下三种因素影响：

节点x是父节点p的左孩子还是右孩子

节点p是不是根节点，如果不是

看节点p是父节点g的左孩子还是右孩子

同时有三种基本操作：

### [](#Zig-Step "Zig Step")Zig Step

![1](http://img.my.csdn.net/uploads/201210/10/1349877565_2986.png)

当p为根节点时，进行zip step操作。

当x是p的左孩子时，对x右旋；

当x是p的右孩子时，对x左旋。  
![](http://img.my.csdn.net/uploads/201210/10/1349877709_4105.png)

### [](#Zig-Zig-Step "Zig-Zig Step")Zig-Zig Step

![](http://img.my.csdn.net/uploads/201210/10/1349877744_7090.png)

当p不是根节点，且x和p不同为左孩子或右孩子时，进行Zig-Zag操作。

当p为左孩子，x为右孩子时，将x左旋后再右旋。

当p为右孩子，x为左孩子时，将x右旋后再左旋。

[](#Advantage "Advantage")Advantage
-----------------------------------

1.可靠的性能——它的平均效率不输于其他平衡树。

2.存储所需的内存少——伸展树无需记录额外的什么值来维护树的信息，相对于其他平衡树，内存占用要小。

3.支持可持久化——可以将其改造成可持久化伸展树。可持久化数据结构允许查询修改之前数据结构的信息，对于一般的数据结构，每次操作都有可能移除一些信息，而可持久化的数据结构允许在任何时间查询到之前某个版本的信息。可持久化这一特性在函数式编程当中非常有用。另外，可持久化伸展树每次一般操作的均摊复杂度是O(log n)

                                 摘自Wikipedia
    

[](#个人理解 "个人理解")个人理解
--------------------

splay是平衡树中一种比较简单易懂的数据结构。

各种操作均摊复杂度都是LogN的

旋转总结起来就是：共线转孙子，不共线转儿子。

我们的目的就是为了使任意时刻树的深度尽量低以保证复杂度。

splay的区间操作比较有意思。

比如我们要对序列区间\[x,y\]搞一些事情，我们可以先把x-1转到root，然后把y+1转到root的儿子

然后我们要求的区间\[x,y\]就是y+1的左子树了。

貌似所有的区间操作都基于这个东西。

splay 还可以维护GSS之类的东西。

splay是动态树（LCT）的基础（这些东西都是tarjan等人发明的 在此致以崇高的敬意orz）。

splay的启发式合并姿势是暴力重构节点数小的那一棵，但是复杂度还是很可观的Nlog2N.

貌似有个东西叫 finger search 合并可以降到NlogN(但是国内没找到相关论文)

splay调试复杂度较高，个人感觉基本的错误在于splaying，Tag的传递还有虚拟点的设置。

(未完待续)

[](#Code们 "Code们")Code们
```cpp

    void rotate(int x,int &k)//旋转
    {
        int y=fa[x],z=fa[y],l,r;
        if(tree[y][0]==x) l=0;else l=1;r=l^1;
        if(y==k) k=x;
        else {
            if(tree[z][0]==y) tree[z][0]=x;//拆. 
            else tree[z][1]=x;
        }
        fa[x]=z,fa[y]=x;fa[tree[x][r]]=y;
        tree[y][l]=tree[x][r];tree[x][r]=y;
        return ;
    }
    void splay(int x,int &k)//splaying
    {
        int y,z;
        while(x!=k)
        {
            y=fa[x],z=fa[y];
            if(y!=k)
            {
                if((tree[y][0]==x)^(tree[z][0]==y)) rotate(x,k);//共线旋转孙子. 
                else rotate(y,k);//不共线旋转儿子. 
            }
            rotate(x,k);
        }
        return ;
    }
    void add(int &k,int x,int f)//加入元素. 
    {
        if(!k){size++;k=size;num[k]=x;fa[k]=f;splay(k,root);return;}
        if(x<num[k]) add(tree[k][0],x,k);//维护二叉树性质.
        else add(tree[k][1],x,k);
        return ;
    }
    void before(int k,int x)//查找前驱.
    {
        if(!k) return ;
        if(num[k]<=x){t1=num[k];x1=k;before(tree[k][1],x);}
        else before(tree[k][0],x);
        return ;
    }
    void after(int k,int x)//查找后继.
    {
        if(!k) return ;
        if(num[k]>=x){t2=num[k];x2=k;after(tree[k][0],x);}
        else after(tree[k][1],x);
        return ;
    }
    void reverse(int x,int y)//区间翻转
    {
        t1=query(x-1+1,root),t2=query(y+1+1,root);
        splay(t1,root),splay(t2,tree[t1][1]);
        rev[tree[t2][0]]^=1;return ;
    }
    void sloveadd(int x,int y)//在x后面添加y个元素(暴力添加最后维护BST性质)
    {
        int k;
        t1=query(x+1,root),t2=query(y+1,root);
        splay(t2,root),splay(t1,tree[t2][0]);
        k=read();
        tree[t1][1]=++tot;fa[tot]=t1;s[tot]=k,size[tot]++,max1[tot]=maxl[tot]=maxr[tot]=k;
        for(int i=1;i<=total-1;i++)
        {
            k=read();fa[++tot]=tot-1;tree[tot-1][1]=tot;
            size[tot]++,sum[tot]=s[tot]=max1[tot]=maxl[tot]=maxr[tot]=k;
        }
        splay(tot,root);
        return ;
    }
    void slovequerysum(int x,int y)//区间求和
    {
        t1=query(x,root),t2=query(y+2,root);
        splay(t1,root),splay(t2,tree[t1][1]);
        printf("%d\n",sum[tree[t2][0]]);
    }
    void slovedelete(int x,int y)//区间删除
    {
        t1=query(x,root),t2=query(y+2,root);
        splay(t1,root),splay(t2,tree[t1][1]);
        tree[t2][0]=0;splay(t2,root);return ;
    }
    void slovechange(int x,int y,int z)//区间修改
    {
        t1=query(x,root),t2=query(y+2,root);
        splay(t1,root),splay(t2,tree[t1][1]);
        int k=tree[t2][0];
        tag[k]=z;s[k]=z;sum[k]=z*size[k];
        splay(k,root);return ;
    }
```