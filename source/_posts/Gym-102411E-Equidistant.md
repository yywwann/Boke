---
title: Gym 102411E Equidistant
tags:
  - BFS
categories:
  - ACM
date: 2019-11-11 13:58:35
---

# 题目

给一棵n个点边权为1的树及树上m个点,让你求一个点,使得所有m个点到那个点的路径相等
n$\leqslant 10^5$

<!--more-->
# 分析

BFS从m个点开始搜索,记录每一层的点数
最后找到一个有m个点同时到达过的点
# 代码
{% fold CODE %}

```c
#include <bits/stdc++.h>
using namespace std;       //     ____   _   _  __   __
#define ll long long       //    / ___| | |_| |   / /
const ll INF = 0x3f3f3f3f; //   | |     |  _  |   V /
const ll N   = 2e5 + 5;    //   | |___  | | | |   | |
const ll MOD = 1e9 + 7;    //    ____| |_| |_|   |_|
ll read() {
  ll   x = 0, f = 1;
  char ch = getchar();

  while (ch < '0' || ch > '9') {
    if (ch == '-') f = -1; ch = getchar();
  }

  while (ch >= '0' && ch <= '9') {
    x = x * 10 + ch - '0'; ch = getchar();
  }
  return x * f;
}

ll  n, m, flg, ans;
int num[N]; // num[i] = 有多少个点同时到达 i 点
int vis[N]; // vis[i] = 最快走几步到达 i 点
vector<ll> g[N];
struct node {
  ll id, fa, step;
};
queue<node> q;

int main() {
  n = read(), m = read();

  for (int i = 1; i < n; i++) {
    int u = read(), v = read();
    g[u].push_back(v);
    g[v].push_back(u);
  }

  for (int i = 1; i <= m; i++) {
    int u = read();
    q.push(node{ u, 0, 1 });
    vis[u] = 1;
    num[u] = 1;
  }

  while (!q.empty()) {
    node u = q.front();
    q.pop();
    ll id = u.id, fa = u.fa, step = u.step;

    for (int i = 0; i < g[id].size(); i++) {
      int v = g[id][i];

      if (v == fa) continue;

      if (vis[v] == 0) { // 如果该点没走过
        vis[v] = step + 1;
        q.push(node{ v, id, step + 1 });
        num[v] += num[id];
      }
      else if (vis[v] == step + 1) { // 可以同时到达该点。
        num[v] += num[id];
      }

      if (num[v] == m) {
        flg = 1, ans = v;
        break;
      }
    }

    if (flg >= 1) break;
  }

  for (int i = 1; i <= n; i++) {
    if (num[i] == m) {
      flg = 1, ans = i;
      break;
    }
  }

  if (flg == 1) cout << "YES" << endl << ans << endl;
  else cout << "NO" << endl;
  return 0;
}
```

{% endfold %}