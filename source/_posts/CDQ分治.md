---
title: CDQ分治学习笔记
date: 2020-05-19 22:55:29
tags: 数据结构
categories: 学习笔记
---


高中时的笔记，重新发布一下qwq。

[](#CDQ分治 "CDQ分治")CDQ分治
=======================

CDQ（**陈丹琦**）分治是一种特殊的分治方法。

它只能处理非强制在线的问题。

CDQ分治在维护一些动态的凸包、半平面交问题也有一定应用，然而本渣渣并不会。

CDQ分治基于时间分治，整体二分基于答案分治。

[](#步骤 "步骤")步骤
--------------

1：将操作按照某个关键字排序

2；算出\[L,mid\]对\[mid+1,R\]的贡献

3；递归处理\[L,mid\]和\[mid+1,R\]

注：这里的区间指的是操作区间。

题目必须满足“修改独立，允许离线”两个条件。

这样的话我们把操作区间二分

会发现后一半的修改操作对前一半的询问操作不会产生影响

前后两个区间的联系只是前一半的修改操作会影响后一半的询问操作。

这个东西我们是可以事先算出来的：对于在满足某种限制下的答案贡献进行合并

用CDQ分治可以解决多维偏序问题

[](#例题 "例题")例题
--------------

### [](#一道简单的题目 "一道简单的题目")一道简单的题目

你有一个长度为N的棋盘，每个格子内有一个整数

两种操作：

1 x A 将格子x里的数字加上A

2 x y输出x y 这个区间内的数字和

1<=N<=100000,操作数不超过10000个，内存限制128M。

是不是很水啊。。。

几个做法

1.暴力O（NM）

2.线段树单点修改区间查询或树状数组维护差分数组O((n+m)logn)

3.分块修改O(1),查询O(q√N)

那么CDQ分治怎么做？

我们对x升序排序，然后按照时间分治，分治的时候记录一个前缀和  
我们要保证贡献的计算不重不漏，根据上面的思路，是不是非常简单啊。。。

#### [](#Code "Code")Code

    #include<algorithm>
    #include<cstdio>
    #define MAXN 100001
    using namespace std;
    int s[MAXN],n,m,tot,t,sum,ans[MAXN];
    struct data{int x,k,t,o,z,belong;}q[MAXN*2],tmp[MAXN*2];
    int read()
    {
        int x=0,f=1;char ch=getchar();
        while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}
        while(ch>='0'&&ch<='9') x=x*10+ch-48,ch=getchar();
        return x*f;
    }
    bool cmp(const data &x,const data &y)
    {
        if(x.x!=y.x) return x.x<y.x;
        else return x.k<y.k;
    }
    void slove(int l,int r)
    {
        if(l==r) return ;
        int mid=(l+r)>>1,ll=l,rr=mid+1;
        for(int i=l;i<=r;i++)
        {
            if(q[i].k==1&&q[i].t<=mid) sum+=q[i].z;
            else if(q[i].k==2&&q[i].t>mid) ans[q[i].belong]+=sum*q[i].z;
        }
        for(int i=l;i<=r;i++)
        {
            if(q[i].t<=mid) tmp[ll++]=q[i];
            else tmp[rr++]=q[i];
        }
        sum=0;
        for(int i=l;i<=r;i++) q[i]=tmp[i];
        slove(l,mid),slove(mid+1,r);
        return ;
    }
    int main()
    {
        int x,y,z;
        n=read();
        for(int i=1;i<=n;i++) x=read(),q[++tot].x=i,q[tot].k=1,q[tot].z=x,q[tot].t=tot;
        m=read();
        for(int i=1;i<=m;i++)
        {
            z=read(),x=read(),y=read();
            if(z&1)  q[++tot].x=x,q[tot].k=1,q[tot].z=y,q[tot].t=tot;
            else
            {
                q[++tot].x=x-1,q[tot].k=2,q[tot].z=-1,q[tot].t=tot,q[tot].belong=++t;
                q[++tot].x=y,q[tot].k=2,q[tot].z=1,q[tot].t=tot,q[tot].belong=t;
            }
        }
        sort(q+1,q+tot+1,cmp);
        slove(1,tot);
        for(int i=1;i<=t;i++) printf("%d\n",ans[i]);
        return 0;
    }
    

### [](#Bzoj-2683-简单题 "Bzoj 2683: 简单题")Bzoj 2683: 简单题

**Time Limit: 50 Sec Memory Limit: 20M.**

Description

你有一个N*N的棋盘，每个格子内有一个整数，初始时的时候全部为0

两种操作：

1 x y A  
1<=x,y<=N，A是正整数 将格子x,y里的数字加上A

2 x1 y1 x2 y2

输出x1 y1 x2 y2这个矩形内的数字和

1<=N<=500000,操作数不超过200000个，内存限制20M。

这道题是CDQ分治的模板题。

如果没有20MB的内存限制的话我们显然可以用二维树状数组或者二维线段树（鬼畜）维护。

如果强制在线的话我们可以用KD-tree去做(蒟蒻并不会)

CDQ分治做法:

我们发现时间，x坐标，y坐标这三个东西的都是有严格顺序要求的。

然后我们考虑怎么去简化问题。

我们先把操作离线，

把一个区间询问拆成4个简单的前缀询问，

然后对于修改和询问操作分别标记。

这样以后对x坐标进行排序。

这样就变成了在时间和y坐标限制下的简化问题

然后我们对于时间进行分治，要注意分治过程中操作时间的顺序的不能乱的。  
在分治过程中，我们可以计算出前一半操作区间中的修改操作对后一半操作区间的询问操作的答案贡献，也就是后一半区间每一个询问操作的答案贡献的一部分。

那关于y坐标怎么搞？

我们可以用一维数据结构来维护它。

我们可以直接维护y坐标所在的这一行的答案贡献。

因为我们已经事先已经保证x坐标升序，这样当我们处理到x坐标所在列的时候，我们发现它前面的列的贡献我们不需要再维护了，因为我们不会再次查询x前面的列数了

这样我们可以直接维护一个矩形的前缀贡献,用树状数组或者线段树维护就好了。

```cpp
    #include<cstdio>
    #include<algorithm>
    #define MAXN 500001
    #define MAXM 200001
    using namespace std;
    int n,s[MAXN],tot,ans[MAXM*4];
    struct data{int x,y,t,o,p,be;}q[MAXM*4],tmp[MAXM*4];
    int read()
    {
        int x=0,f=1;char ch=getchar();
        while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}
        while(ch>='0'&&ch<='9') x=x*10+ch-48,ch=getchar();
        return x*f;
    }
    void add(int x,int z)
    {
        while(x<=n) s[x]+=z,x+=x&-x;
        return ;
    }
    int getsum(int x)
    {
        int sum=0;
        while(x>0) sum+=s[x],x-=x&-x;
        return sum;
    }
    bool cmp(const data &x,const data &y)
    {
        if(x.x==y.x&&x.y==y.y) return x.p<y.p;
        if(x.x==y.x) return x.y<y.y;
        return x.x<y.x;
    }
    void slove(int l,int r)
    {
        if(l==r) return ;
        int mid=(l+r)>>1;
        for(int i=l;i<=r;i++)
        {
            if(q[i].t<=mid&&q[i].p==1) add(q[i].y,q[i].o);
            else if(q[i].t>mid&&q[i].p==2) ans[q[i].be]+=getsum(q[i].y)*q[i].o;
        }
        for(int i=l;i<=r;i++)
          if(q[i].t<=mid&&q[i].p==1) add(q[i].y,-q[i].o);
        int ll=l,rr=mid+1;
        for(int i=l;i<=r;i++)
        {
            if(q[i].t<=mid) tmp[ll++]=q[i];
            else tmp[rr++]=q[i];
        }
        for(int i=l;i<=r;i++) q[i]=tmp[i];
        slove(l,mid),slove(mid+1,r);
        return ;
    }
    int main()
    {
        n=read();
        int z,x1,y1,x2,y2,k,t=0;
        while(true)
        {
            z=read();
            if(z==1){
                x1=read(),y1=read(),k=read();
                q[++tot].x=x1,q[tot].y=y1,q[tot].o=k,q[tot].p=1,q[tot].t=tot;
            }
            else if(z==2)
            {
                x1=read(),y1=read(),x2=read(),y2=read();
                q[++tot].x=x1-1,q[tot].y=y1-1,q[tot].o=1,q[tot].p=2,q[tot].t=tot,q[tot].be=++t;
                q[++tot].x=x1-1,q[tot].y=y2,q[tot].o=-1,q[tot].p=2,q[tot].t=tot,q[tot].be=t;
                q[++tot].x=x2,q[tot].y=y1-1,q[tot].o=-1,q[tot].p=2,q[tot].t=tot,q[tot].be=t;
                q[++tot].x=x2,q[tot].y=y2,q[tot].o=1,q[tot].p=2,q[tot].t=tot,q[tot].be=t;
            }
            else break;
        }
        sort(q+1,q+tot+1,cmp);
        slove(1,tot);
        for(int i=1;i<=t;i++) printf("%d\n",ans[i]);
        return 0;
    }
```

[](#参考文献 "参考文献")参考文献
--------------------

许昊然 2013年国家集训队论文《浅谈数据结构题的几个非经典题非经典算法》