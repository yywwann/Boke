---
title: 小w的糖果
tags:
  - 差分
  - 前缀和
categories:
  - ACM
date: 2019-06-24 15:58:02
---

# 差分 前缀和
[题目](https://ac.nowcoder.com/acm/contest/923/C)
> 链接：https://ac.nowcoder.com/acm/contest/923/C
来源：牛客网
1、如果某轮发糖果的是tokitsukaze，她将会从一个位置pos开始，依次向右给每个人1个糖果。
2、如果某轮发糖果的是teito，他将会从一个位置pos开始，依次向右，他将会给他碰到的第一个人发1个糖果，给他碰到的第二个人发2个糖果，给他碰到的第三个人发3个糖果...碰到的第k个人发k个糖果，直到向右走到编号为n的人为止。
3、如果某轮发糖果的是winterzz1，众所周知小w是个大方的人，所以他发的糖最多，他将会从一个位置pos开始，依次向右，它将会给他碰到的第一个人发1个糖果，给他碰到的第二个人发4个糖果，给他碰到的第三个人发9个糖果...碰到的第k个人发$k^2$个糖果直到向右走到编号为n的人为止。
<!--more-->

# 分析
作者：四糸智乃
链接：https://ac.nowcoder.com/discuss/200054?type=101&order=0&pos=6&page=1
来源：牛客网

有两种思路：

第一种思路：离线处理，利用多项式，构造函数f(x)=ax2+bx+c，只用保存a,b,c三个值，然后我们只要维护a,b,c三个系数，就能知道f(i)的值，碰到修改操作的时候只要修改a,b,c的值即可。

第二种思路：还是先离线，这次利用前缀和与差分数组的关系。我们知道区间加常数c可以先做差分，此时操作变为单点修改，然后使用前缀和还原。

区间加一次函数bx+c呢，则可以做二阶差分操作（先差分一次，再对差分数组做一次差分）这时区间加一次函数bx+c就变为了单点修改，然后做二阶前缀和（前缀和的前缀和）还原，然后这个题就做完了。

区间加二次函数ax2+bx+c 以此类推可以做三阶差分，也就是做差分，然后再差分，再差分。最后做三次前缀和还原。
https://ac.nowcoder.com/acm/contest/view-submission?submissionId=40782126
说点题外话，这个操作是可以扩展的。

k次多项式的k+1阶差分可以用k+1个常数来表示，并且这k+1个常数的和总是等于最高项系数乘以k的阶乘。这个性质我在之前的牛客练习赛中有出过题目：https://ac.nowcoder.com/acm/contest/308/F
# 代码
{% fold CODE1 %}

```c
#include<bits/stdc++.h>
using namespace std;
const int MAXN=100005;
const long long mod=(long long)1e9+7;
long long d1[MAXN],d2[MAXN],d3[MAXN];//1 id id2
int n,m,T,type,pos;
void pre_sum(long long a[])
{
    for(int i=1;i<=n;++i)
    {
        a[i]=(a[i]+a[i-1])%mod;
    }
    return;
}
int main()
{
    scanf("%d",&T);
    while(T--)
    {
        memset(d1,0,sizeof(d1));
        memset(d2,0,sizeof(d2));
        memset(d3,0,sizeof(d3));
        scanf("%d %d",&n,&m);
        while(m--)
        {
            scanf("%d %d",&type,&pos);
            if(type==1)
            {
                d1[pos]++;
            }
            if(type==2)
            {
                d2[pos]++;
            }
            if(type==3)
            {
                d3[pos]++;
                d3[pos+1]++;
            }
        }
        pre_sum(d1);
 
        pre_sum(d2);
        pre_sum(d2);
 
        pre_sum(d3);
        pre_sum(d3);
        pre_sum(d3);
 
        for(int i=1;i<=n;++i)
        {
            printf("%lld%c",(d1[i]+d2[i]+d3[i])%mod,i==n?'\n':' ');
        }
    }
    return 0;
}
```


{% endfold %}

