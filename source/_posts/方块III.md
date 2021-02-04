---
title: 方块III
tags:
  - 线段树
categories:
  - ACM
date: 2019-06-21 08:48:59
---

# 区间最值 区间赋值
[题面](https://ac.nowcoder.com/acm/problem/14601)
> 有 N 个方块排成一排，每个方块都染有颜色，第 i 个的颜色为 Ci，一共有M种颜色，每种颜色的方块都有不同的价值Wi，现在要求找出一段方块，使得其中只出现过一次的方块的价值总和最大。

<!--more-->
# 分析
对于同一种颜色，每次只能取一段。
按顺序遍历，假设取此颜色该区间，将此颜色前一段区间删去，给这一段区间赋值。
然后每一次都取最大值

# 代码
{% fold CODE %}

```c
#include <bits/stdc++.h>
using namespace std;       //     ____   _   _  __   __
#define ll long long       //    / ___| | |_| |   / /
const ll INF = 0x3f3f3f3f; //   | |     |  _  |   V /
const ll N   = 1e5 + 5;    //   | |___  | | | |   | |
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

struct segt {
  ll *a;
  struct Tree {
    int l, r;
    ll  sum, lz;
    void update(ll v) {
      sum += v;
      lz  += v;
    }
  } tree[N * 4];

  void modify(ll *arr) {
    a = arr;
  }

  void pushup(int x) {
    tree[x].sum = max(tree[2 * x].sum, tree[2 * x + 1].sum);
  }

  void pushdown(int x) {
    if (tree[x].lz != 0) {
      tree[2 * x].update(tree[x].lz);
      tree[2 * x + 1].update(tree[x].lz);
      tree[x].lz = 0;
    }
  }

  void build(int x, int l, int r) {
    tree[x].l  = l;
    tree[x].r  = r;
    tree[x].lz = 0;


    if (l == r) {
      tree[x].sum = a[l];
      return;
    }
    int mid = (l + r) / 2;
    build(2 * x,     l,       mid);
    build(2 * x + 1, mid + 1, r);
    pushup(x); // 如果在建树的过程中给sum赋值，记得后面要pushup
  }

  void update(int x, int l, int r, ll c) {
    int L = tree[x].l, R = tree[x].r;
    int mid = (L + R) / 2;

    if ((l <= L) && (r >= R)) {
      tree[x].update(c);
      return;
    }

    pushdown(x);

    if (l <= mid) update(2 * x, l, r, c);

    if (r > mid) update(2 * x + 1, l, r, c);
    pushup(x);
  }

  ll query(int x, int l, int r) {
    int L = tree[x].l, R = tree[x].r;
    ll  mid = (L + R) / 2;
    ll  res = 0;

    if ((l <= L) && (r >= R)) { // 要更新区间包括了该区间
      return tree[x].sum;
    }

    pushdown(x);

    if (l <= mid) res = max(res, query(2 * x, l, r));

    if (r > mid) res = max(res, query(2 * x + 1, l, r));
    pushup(x);
    return res;
  }
} tree;


int n, m;
ll  c[N], w[N];
vector<int> v[N];
ll ans;
ll a[N];
int main() {
  while (cin >> n >> m) {
    ans = 0;
    tree.modify(a);
    tree.build(1, 1, n);

    for (int i = 1; i <= n; i++) c[i] = read(), v[i].clear();

    for (int i = 1; i <= m; i++) w[i] = read();


    for (int i = 1; i <= n; i++) {
      if (v[c[i]].size() == 0) {
        tree.update(1, 1, i, w[c[i]]);
      }
      else if (v[c[i]].size() == 1) {
        tree.update(1, 1,              v[c[i]][0], -w[c[i]]);
        tree.update(1, v[c[i]][0] + 1, i,          w[c[i]]);
      }
      else {
        tree.update(1,
                    v[c[i]][v[c[i]].size() - 2] + 1,
                    v[c[i]][v[c[i]].size() - 1] + 1,
                    -w[c[i]]);
        tree.update(1,
                    v[c[i]][v[c[i]].size() - 1] + 1,
                    i,
                    w[c[i]]);
      }
      v[c[i]].push_back(i);
      ans = max(ans, tree.query(1, 1, n));
    }
    cout << ans << endl;
  }
  return 0;
}
```

{% endfold %}