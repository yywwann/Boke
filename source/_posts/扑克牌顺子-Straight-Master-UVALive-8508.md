---
title: UVALive 8508 Straight Master 扑克牌顺子
date: 2018-09-02 16:33:34
tags:
 - 贪心
categories:
 - ACM
---


[Straight Master UVALive - 8508](https://vjudge.net/problem/UVALive-8508)
# 题意
给我们一堆牌，判断牌能否只用顺子全部打出。

<!--more-->
#  {% fold CODE %} 
```C
        cin>>n;
        for(int i = 1; i <= n; i++){
            // int x;
            // cin>>x;
            // tag[x]++;
            cin>>tag[i];
        }
        for(int i = N - 1; i >= 1; i--){
            tag[i] -= tag[i - 1];
        }
        int p = 0;
        for(int i = 1; i < N; i++){
            if(tag[i] < 0){
                int need = -tag[i];
                while(need && i - p >= 3){
                    int good = min(need,tag[p]);
                    need -= good;
                    tag[p] -= good;
                    if(!tag[p])p++;
                }
                tag[i] = need;
            }
        }
        bool f = 0;
        for(int i = 1; i < N; i++)if(tag[i])f = 1;
        if(f)cout<<"NO\n";
        else cout<<"YES\n";
```
{% endfold %}
