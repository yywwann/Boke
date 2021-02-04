---
title: MAC Atom C++ 环境搭建
date: 2018-11-01 17:16:53
tags:
categories:
---
# 前提
MACOS
Xcode
# Atom下载地址
[https://atom.io/](https://atom.io/)
# 插件下载

| Name | deccribe |
| --- | --- |
| ~~autocomplete-clang~~  | autocomplete for C/C++/ObjC using clang |
| ~~build~~ | Build your current project, directly from Atom |
| ~~linter~~ | A Base Linter with Cow Powers to visualize errors and other kind-of messages, easily |
| ~~linter-clang~~ | A linter plugin for Linter provides an interface to clang |
| ~~script~~ | Run code in Atom! |
| minimap | A preview of the full source code. 代码略缩图|
| gpp-compiler | Compile and run C and C++ within Atom c++代码编译运行|
| active-power-mode | Active POWER MODE to write your code in style. 炫酷动画|
| Atom Beautify | 代码美化|
|Docblockr|代码注释优化|
|Highlight Selected|选中高亮|
|Emmet|对 html 代码各种快捷操作|
|Highlight Line|当前行高亮|
|Simplified Chinese Menu|简体中文汉化包|
|Autoclose Html||
|File Icons|优化文件图标显示|
|Pigments|颜色#FFFFFF代码显示|
|Platformio Ide Terminal|内置终端|
|Uncrustify|代码格式 配合Atom Beautify|

<!--more-->
# 操作
F5编译+运行
# 备注
修改默认tap长度为4

# Gpp Compiler
C++ Compiler Options
-std=c++11

# Snippets

{% fold snippets.cson %}

```json
# Your snippets
#
# Atom snippets allow you to enter a simple prefix in the editor and hit tab to
# expand the prefix into a larger code block with templated values.
#
# You can create a new snippet in this file by typing "snip" and then hitting
# tab.
#
# An example CoffeeScript snippet to expand log to console.log:
#
# '.source.coffee':
#   'Console log':
#     'prefix': 'log'
#     'body': 'console.log $1'
#
# Each scope (e.g. '.source.coffee' above) can only be declared once.
#
# This file uses CoffeeScript Object Notation (CSON).
# If you are unfamiliar with CSON, you can read more about it in the
# Atom Flight Manual:
# http://flight-manual.atom.io/using-atom/sections/basic-customization/#_cson

'.source.cpp':
  'python coding':
    'prefix': 'acm'
    'body': """
#include <bits/stdc++.h>
using namespace std;    //     ____   _   _  __   __
#define ll long long    //    / ___| | |_| |   / /
const ll INF = 2e18;    //   | |     |  _  |   V /
const ll N   = 1e5 + 5; //   | |___  | | | |   | |
const ll MOD = 1e9 + 7; //    ____| |_| |_|   |_|
ll read() {
  ll x = 0, f = 1; char ch = getchar();
  for (; ch < '0' || ch > '9'; ch = getchar()) if (ch == '-') f = -1;
  for (; ch >= '0' && ch <= '9'; ch = getchar()) x = x * 10 + ch - '0';
  return x * f;
}

${1}

int main() {

  return 0;
}"""

  '线段树':
    'prefix': 'fseg'
    'body':"""
struct segt {
  ll *a;
  ll  SUM, MAX, MIN;
  struct Tree {
    int l, r;
    ll  sum, lz, max, min, set;
    void update(ll v) {
      sum += v * (r - l + 1);
      lz  += v;
      max += v;
      min += v;
    }

    void setval(ll v) {
      sum = v * (r - l + 1);
      lz  = 0;
      max = v;
      min = v;
      set = v;
    }
  } tree[N * 4];

  void modify(ll *arr) {
    a = arr;
  }

  void pushup(int x) {
    tree[x].sum = tree[2 * x].sum + tree[2 * x + 1].sum;
    tree[x].max = max(tree[2 * x].max, tree[2 * x + 1].max);
    tree[x].min = min(tree[2 * x].min, tree[2 * x + 1].min);
  }

  void pushdown(int x) {
    if (tree[x].set != -1) {
      tree[2 * x].setval(tree[x].set);
      tree[2 * x + 1].setval(tree[x].set);
      tree[x].set = -1;
    }
    if (tree[x].lz != 0) {
      tree[2 * x].update(tree[x].lz);
      tree[2 * x + 1].update(tree[x].lz);
      tree[x].lz = 0;
    }
  }

  // 建树
  void build(int x, int l, int r) {
    tree[x].l   = l;
    tree[x].r   = r;
    tree[x].sum = tree[x].max = tree[x].min = tree[x].lz = 0;
    tree[x].set = -1;
    if (l == r) {
      return;
    }
    int mid = (l + r) / 2;
    build(2 * x,     l,       mid);
    build(2 * x + 1, mid + 1, r);
    pushup(x);
  }

  // 将 l-r 范围的数都加上 c
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

  // 将 l-r 范围的数变成 c， 如果 set 为 -1 表示没有被赋值过
  void setval(int x, int l, int r, ll c) {
    int L = tree[x].l, R = tree[x].r;
    int mid = (L + R) / 2;
    if ((l <= L) && (r >= R)) {
      tree[x].setval(c);
      return;
    }

    pushdown(x);
    if (l <= mid) setval(2 * x, l, r, c);
    if (r > mid) setval(2 * x + 1, l, r, c);
    pushup(x);
  }

  // 查询区间和
  ll querySUM(int x, int l, int r) {
    int L = tree[x].l, R = tree[x].r;
    int mid = (L + R) / 2;
    ll  res = 0;
    if ((l <= L) && (r >= R)) { // 要更新区间包括了该区间
      return tree[x].sum;
    }

    pushdown(x);
    if (l <= mid) res += querySUM(2 * x, l, r);
    if (r > mid) res += querySUM(2 * x + 1, l, r);
    return res;
  }

  // 查询区间最大值
  ll queryMAX(int x, int l, int r) {
    int L = tree[x].l, R = tree[x].r;
    int mid = (L + R) / 2;
    ll  res = -INF;
    if ((l <= L) && (r >= R)) { // 要更新区间包括了该区间
      return tree[x].max;
    }

    pushdown(x);
    if (l <= mid) res = max(res, queryMAX(2 * x, l, r));
    if (r > mid) res = max(res, queryMAX(2 * x + 1, l, r));
    return res;
  }

  // 查询区间最小值
  ll queryMIN(int x, int l, int r) {
    int L = tree[x].l, R = tree[x].r;
    int mid = (L + R) / 2;
    ll  res = INF;
    if ((l <= L) && (r >= R)) { // 要更新区间包括了该区间
      return tree[x].min;
    }

    pushdown(x);
    if (l <= mid) res = min(res, queryMIN(2 * x, l, r));
    if (r > mid) res = min(res, queryMIN(2 * x + 1, l, r));
    return res;
  }

  // 一次性查询区间 SUM，MAX，MIN
  void query(int x, int l, int r) {
    int L = tree[x].l, R = tree[x].r;
    int mid = (L + R) / 2;
    if ((l <= L) && (r >= R)) { // 要更新区间包括了该区间
      SUM += tree[x].sum;
      MAX  = max(MAX, tree[x].max);
      MIN  = min(MIN, tree[x].min);
      return;
    }

    pushdown(x);
    if (l <= mid) query(2 * x, l, r);
    if (r > mid) query(2 * x + 1, l, r);
  }
} tree;
"""

  'for 1 - n':
    'prefix': 'fn'
    'body':"""
for (int ${1:i} = ${2:1}; ${1:i} <= ${3:n}; ${1:i}++) ${4}
"""

  'bound found':
    'prefix': 'fef'
    'body':"""
ll ans = 0, l = 1, r = n;
while (l <= r) {
  ll mid = (l + r) / 2;
  if (check(mid)) ans = r = mid - 1;
  else l = mid + 1;
}
"""
  'push_back':
    'prefix': 'pb'
    'body':"""
push_back($1);
"""
  'PowerMod':
    'prefix': 'fpow'
    'body':"""
ll PowerMod(ll a,ll b,ll c){
	ll ans = 1;
	a = a % c;
	while(b > 0){
		if(b % 2 == 1) ans = (ans * a) % c;
		b = b / 2;
		a = (a * a) % c;
	}
	return ans;
}
"""

  'gcd':
    'prefix': 'fgcd'
    'body':"""
ll gcd(ll a, ll b) {
  return b == 0 ? a : gcd(a, b % a);
}

ll lcm(ll a, ll b) {
  return a / gcd(a, b) * b;
}
"""

  'T input':
    'prefix': 'tin'
    'body':"""
for (int _ = read(); _; _--) {
  $1
}
"""
  '树的直径':
    'prefix': 'ftdia'
    'body':"""
struct node {
  int u, d;
  node() {}
  node(int u0, int d0) : u(u0), d(d0) {}
  bool operator<(const node& a) const {
    return d > a.d; // 最小值优先
  }
};
int  Max, k;
bool vis[N];

vector<node> g[N];


int bfs(int x) {
  priority_queue<node> q;
  q.push(node(x, 0));
  Max = k = 0;
  memset(vis, false, sizeof(vis));
  vis[x] = true;
  while (!q.empty()) {
    node u = q.top();
    q.pop();
    for (int i = 0; i < g[u.u].size(); i++) {
      node v = g[u.u][i];
      if (!vis[v.u]) {
        vis[v.u] = true;
        q.push(node(v.u, u.d + v.d));
      }
    }
    if (u.d > Max) Max = u.d, k = u.u;
  }
  return k;
}
"""
  'KMP':
    'prefix': 'fkmp'
    'body':"""
string Find, a;
int    NEXT[N];

void getNEXT() {
  int i = 0, j = NEXT[0] = -1;
  while (i < Find.size()) {
    if ((j == -1) || (Find[i] == Find[j])) {
      i++; j++;
      NEXT[i] = j;
    }
    else {
      j = NEXT[j];
    }
  }
}

int ManyKmp() {
  int i = 0, j = 0, ans = 0;
  while (i < a.size()) {
    if ((j == -1) || (a[i] == Find[j])) {
      i++, j++;
      if (j == Find.size()) {
        ans++;
        j = NEXT[j];
      }
    }
    else j = NEXT[j];
  }
  return ans;
} // 在a中找几个可以重叠的Find

int FirstKmp() {
  int i = 0, j = 0;
  while (i < a.size()) {
    if ((j == -1) || (a[i] == Find[j])) {
      i++, j++;
      if (j == Find.size()) {
        return i - j;
      }
    }
    else j = NEXT[j];
  }
  return -1;
} // 在a中找到第一个Find的位置

int ManyKMP() {
  getNEXT();
  int i = 0, j = 0, ans = 0;
  while (i < a.size()) {
    if ((j == -1) || (a[i] == Find[j])) {
      i++, j++;
      if (j == Find.size()) {
        ans++;
        j = 0;
      }
    }
    else j = NEXT[j];
  }
  return ans;
} // 在a中找几个不可以重叠的Find

// 如果一个字符串满足 len%(len-next[len])==0，则这个字符串是一个周期串（存在最小循环节），循环次数为 len / (
// len-next[len] )
// 如果 len%（len-next[len]）!=0,这个字符串也存在循环节，但是最后一个循环节不完整

"""
  '树链剖分 边权':
    'prefix': 'fslpf'
    'body':"""
ll n;
vector<int> g[maxn];
struct Edge {
  ll u, v, w;
};
vector<Edge> edge, que;
ll f[maxn];    // 父亲
ll d[maxn];    // 深度
ll son[maxn];  // 重儿子
ll size[maxn]; // 大小
ll top[maxn];  // 所在链的顶端
ll id[maxn];   // dfs序
ll rk[maxn];   // dfs序对应的结点编号
ll cnt;        // dfs的时间戳



void dfs1(ll x, ll pre, ll deep) {
  size[x] = 1, d[x] = deep;
  f[x]    = pre;
  for (int i = 0; i < g[x].size(); i++) {
    int y = g[x][i];
    if (y == pre) continue;
    f[y] = x; dfs1(y, x, deep + 1); size[x] += size[y];
    if (size[son[x]] < size[y]) son[x] = y;
  }

  // for (ll i = head[x]; i != -1; i = edge[i].nxt) {
  //   ll y = edge[i].to;
  //   if (y == pre) continue;
  //   f[y] = x; dfs1(y, x, deep + 1); size[x] += size[y];
  //   if (size[son[x]] < size[y]) son[x] = y;
  // }
}

void dfs2(ll x, ll tp) {
  top[x] = tp; id[x] = ++cnt; rk[cnt] = x;
  if (son[x]) dfs2(son[x], tp);
  for (int i = 0; i < g[x].size(); i++) {
    int y = g[x][i];
    if ((y != son[x]) && (y != f[x])) dfs2(y, y);
  }

  // for (ll i = head[x]; i != -1; i = edge[i].nxt) {
  //   ll y = edge[i].to;
  //   if ((y != son[x]) && (y != f[x])) dfs2(y, y);
  // }
}

struct segt {
  ll *a;
  ll  SUM, MAX, MIN;
  struct Tree {
    int l, r;
    ll  sum, lz, max, min, set;
    void update(ll v) {
      sum += v * (r - l + 1);
      lz  += v;
      max += v;
      min += v;
    }

    void setval(ll v) {
      sum = v * (r - l + 1);
      lz  = 0;
      max = v;
      min = v;
      set = v;
    }
  } tree[maxn * 4];

  void modify(ll *arr) {
    a = arr;
  }

  void pushup(int x) {
    tree[x].sum = tree[2 * x].sum + tree[2 * x + 1].sum;
    tree[x].max = max(tree[2 * x].max, tree[2 * x + 1].max);
    tree[x].min = min(tree[2 * x].min, tree[2 * x + 1].min);
  }

  void pushdown(int x) {
    if (tree[x].set != -1) {
      tree[2 * x].setval(tree[x].set);
      tree[2 * x + 1].setval(tree[x].set);
      tree[x].set = -1;
    }
    if (tree[x].lz != 0) {
      tree[2 * x].update(tree[x].lz);
      tree[2 * x + 1].update(tree[x].lz);
      tree[x].lz = 0;
    }
  }

  // 建树
  void build(int x, int l, int r) {
    tree[x].l   = l;
    tree[x].r   = r;
    tree[x].sum = tree[x].max = tree[x].min = tree[x].lz = 0;
    tree[x].set = -1;
    if (l == r) {
      tree[x].sum = tree[x].max = tree[x].min = 1000000;
      return;
    }
    int mid = (l + r) / 2;
    build(2 * x,     l,       mid);
    build(2 * x + 1, mid + 1, r);
    pushup(x);
  }

  // 将 l-r 范围的数都加上 c
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

  // 将 l-r 范围的数变成 c， 如果 set 为 -1 表示没有被赋值过
  void setval(int x, int l, int r, ll c) {
    int L = tree[x].l, R = tree[x].r;
    int mid = (L + R) / 2;
    if ((l <= L) && (r >= R)) {
      tree[x].setval(c);
      return;
    }

    pushdown(x);
    if (l <= mid) setval(2 * x, l, r, c);
    if (r > mid) setval(2 * x + 1, l, r, c);
    pushup(x);
  }

  // 查询区间和
  ll querySUM(int x, int l, int r) {
    int L = tree[x].l, R = tree[x].r;
    int mid = (L + R) / 2;
    ll  res = 0;
    if ((l <= L) && (r >= R)) { // 要更新区间包括了该区间
      return tree[x].sum;
    }

    pushdown(x);
    if (l <= mid) res += querySUM(2 * x, l, r);
    if (r > mid) res += querySUM(2 * x + 1, l, r);
    return res;
  }

  // 查询区间最大值
  ll queryMAX(int x, int l, int r) {
    int L = tree[x].l, R = tree[x].r;
    int mid = (L + R) / 2;
    ll  res = -INF;
    if ((l <= L) && (r >= R)) { // 要更新区间包括了该区间
      return tree[x].max;
    }

    pushdown(x);
    if (l <= mid) res = max(res, queryMAX(2 * x, l, r));
    if (r > mid) res = max(res, queryMAX(2 * x + 1, l, r));
    return res;
  }

  // 查询区间最小值
  ll queryMIN(int x, int l, int r) {
    int L = tree[x].l, R = tree[x].r;
    int mid = (L + R) / 2;
    ll  res = INF;
    if ((l <= L) && (r >= R)) { // 要更新区间包括了该区间
      return tree[x].min;
    }

    pushdown(x);
    if (l <= mid) res = min(res, queryMIN(2 * x, l, r));
    if (r > mid) res = min(res, queryMIN(2 * x + 1, l, r));
    return res;
  }

  // 一次性查询区间 SUM，MAX，MIN
  void query(int x, int l, int r) {
    int L = tree[x].l, R = tree[x].r;
    int mid = (L + R) / 2;
    if ((l <= L) && (r >= R)) { // 要更新区间包括了该区间
      SUM += tree[x].sum;
      MAX  = max(MAX, tree[x].max);
      MIN  = min(MIN, tree[x].min);
      return;
    }

    pushdown(x);
    if (l <= mid) query(2 * x, l, r);
    if (r > mid) query(2 * x + 1, l, r);
  }
} tree;

void updates(ll x, ll y, ll c) {
  while (top[x] != top[y]) {
    if (d[top[x]] < d[top[y]]) swap(x, y);
    tree.setval(1, id[top[x]], id[x], c);
    x = f[top[x]];
  }
  if (id[x] > id[y]) swap(x, y);
  if (x != y) tree.setval(1, id[x] + 1, id[y], c);
}

ll Min(ll x, ll y) {
  ll res = INF;
  while (top[x] != top[y]) {
    if (d[top[x]] < d[top[y]]) swap(x, y);
    res = min(res, tree.queryMIN(1, id[top[x]], id[x]));
    x = f[top[x]];
  }
  if (id[x] > id[y]) swap(x, y);
  if (x != y) res = min(res, tree.queryMIN(1, id[x] + 1, id[y]));
  return res;
}
"""
  'LCA':
    'prefix': 'flca'
    'body':"""
struct Node {
  int to, next;
};
Node edge1[N << 1];
int  head1[N << 1], d[N], fa[N][30], tot = 0;

inline void add(int x, int y) {
  edge1[++tot].to = y;
  edge1[tot].next = head1[x];
  head1[x]         = tot;
}

inline void build(int u, int pre) {
  d[u] = d[pre] + 1; fa[u][0] = pre;
  for (int i = 0; fa[u][i]; ++i) fa[u][i + 1] = fa[fa[u][i]][i];
  for (int i = head1[u]; i; i = edge1[i].next) {
    int e = edge1[i].to;
    if (e != pre) build(e, u);
  }
}

inline int lca(int u, int v) {
  if (d[u] > d[v]) swap(u, v);
  for (int i = 20; i >= 0; --i) {
    if (d[u] <= d[v] - (1 << i)) {
      v = fa[v][i];
    }
  }
  if (u == v) return u;
  for (int i = 20; i >= 0; --i) {
    if (fa[u][i] != fa[v][i]) {
      u = fa[u][i], v = fa[v][i];
    }
  }
  return fa[u][0];
}
"""
  'SPFA':
    'prefix': 'fspfa'
    'body':"""
ll n, m, cnt = 0;
ll dis[N], vis[N], head[N];
ll pre[N], pre2[N];
struct node {
  ll next, to, w;
} x[N];
void add(ll start, ll endd, ll w) {
  x[cnt].to   = endd;
  x[cnt].w    = w;
  x[cnt].next = head[start];
  head[start] = cnt++;
}

void init() {
  cnt = 0;
  memset(head, -1, sizeof head);
  memset(x,     0, sizeof x);
  memset(pre,   0, sizeof pre);
  memset(pre2,  0, sizeof pre2);
}

void spfa(int st, ll *dis) {
  queue<ll> q;
  for (ll i = 1; i <= n; i++) {
    dis[i] = INF; // 初始化
    vis[i] = 0;
  }
  q.push(st);
  dis[st] = 0;
  vis[st] = 1;                                      // 第一个顶点入队，进行标记
  while (!q.empty()) {
    ll u = q.front();                              // 取出队首
    q.pop();
    vis[u] = 0;                                    // 出队标记
    for (ll i = head[u]; i != -1; i = x[i].next) { // 找这条边的同起点的上一条边
      ll v = x[i].to;
      if (dis[v] > dis[u] + x[i].w) {              // 如果有最短路就更改
        dis[v] = dis[u] + x[i].w;
        if (vis[v] == 0) {                         // 未入队则入队
          vis[v] = 1;                              // 标记入队
          q.push(v);
        }
        pre[v]  = u;
        pre2[v] = i;
      }
    }
  }
}
"""
  'Dijkstra':
    'prefix': 'fdij'
    'body':"""
struct node {
  ll v, c;
  bool operator<(const node& r) const {
    return c > r.c;
  }
};
vector<node> E[N];
ll vis[N], dis[N];
void Dijkstra(int n, int start, ll *dis) { // 点的编号从1开始
  for (int i = 1; i <= n; i++) {
    dis[i] = INF;
    vis[i] = 0;
  }
  priority_queue<node> q;
  while (!q.empty()) q.pop();
  dis[start] = 0;
  q.push(node{ start, 0 });
  while (!q.empty()) {
    node tmp = q.top();
    q.pop();
    int u = tmp.v;
    if (vis[u]) continue;
    vis[u] = true;
    for (int i = 0; i < E[u].size(); i++) {
      int v = E[u][i].v;
      int c = E[u][i].c;
      if (!vis[v] && (dis[v] > dis[u] + c)) {
        dis[v] = dis[u] + c;
        q.push(node{ v, dis[v] });
      }
    }
  }
}

void addedge(int u, int v, int w) {
  E[u].push_back(node{ v, w });
}

void init() {
  for (int i = 0; i < N; i++) E[i].clear();
}
"""
  'ST表':
    'prefix': 'fst'
    'body':"""
int stmin[N][20], stmax[N][20];
void InitSt()
{
  for (int j = 1; (1 << j) < n; j++)
  {
    for (int i = 0; i + (1 << j) - 1 < n; i++)
    {
      stmin[i][j] = min(stmin[i][j - 1], stmin[i + (1 << (j - 1))][j - 1]);

      // stmax[i][j] = max(stmax[i][j-1], stmax[i+(1<<(j-1))][j-1]);
    }
  }
}

int query(int l, int r)
{
  int x = (int)(log(double(r - l + 1) / log(2.0)));
  return min(stmin[l][x], stmin[r - (1 << x) + 1][x]); // 找最大值改成max
}
"""
  '优先队列':
    'prefix': 'fpri'
    'body':"""
struct node {
  ll x, y;
  node(ll _x, ll _y) {
    x = _x, y = _y;
  }

  friend bool operator<(node a, node b)
  {
    // return a.x > b.x;//结构体中，x小的优先级高
    return a.x < b.x; // 结构体中，x大的优先级高
  }
};
priority_queue<node> q4;
"""
  '马拉车':
    'prefix': 'fman'
    'body':"""
string Manacher(string s) {
  /*改造字符串*/
  string res = "$#";
  for (int i = 0; i < s.size(); ++i) {
    res += s[i];
    res += "#";
  }

  /*数组*/
  vector<int> P(res.size(), 0);
  int mi = 0, right = 0;                                  // mi为最大回文串对应的中心点，right为该回文串能达到的最右端的值
  int maxLen = 0, maxPoint = 0;                           // maxLen为最大回文串的长度，maxPoint为记录中心点
  for (int i = 1; i < res.size(); ++i) {
    P[i] = right > i ? min(P[2 * mi - i], right - i) : 1; // 关键句，文中对这句以详细讲解
    while (res[i + P[i]] == res[i - P[i]]) ++P[i];
    if (right < i + P[i]) {                               // 超过之前的最右端，则改变中心点和对应的最右端
      right = i + P[i];
      mi    = i;
    }
    if (maxLen < P[i]) { // 更新最大回文串的长度，并记下此时的点
      maxLen   = P[i];
      maxPoint = i;
    }
  }
  return s.substr((maxPoint - maxLen) / 2, maxLen - 1);
}
"""
  '素数筛':
    'prefix': 'fprime'
    'body':"""
bool isprime[N];                                  // isprime[i]表示i是不是质数
int  prime[N];                                    // prime[N]用来存质数  从0开始
int  tot;                                         // tot表示[2，N]之间质数的数量
void init() {
  for (int i = 2; i < N; i++) isprime[i] = true;  // 初始化为质数
  for (int i = 2; i < N; i++) {
    if (isprime[i]) prime[tot++] = i;             // 把质数存起来
    for (int j = 0; j < tot && i * prime[j] < N; j++) {
      isprime[i * prime[j]] = false;              // 质数的倍数一定不是质数
      if (i % prime[j] == 0) break;               // 保证每个合数被它最小的质因数筛去
    }
  }
}
"""
  '高精度':
    'prefix': 'fhigh'
    'body':"""
#include <bits/stdc++.h>
using namespace std;    //     ____   _   _  __   __
#define ll long long    //    / ___| | |_| |   / /
const ll INF = 2e18;    //   | |     |  _  |   V /
const ll N   = 1e5 + 5; //   | |___  | | | |   | |
const ll MOD = 1e9 + 7; //    ____| |_| |_|   |_|
ll read() {
  ll x = 0, f = 1; char ch = getchar();
  for (; ch < '0' || ch > '9'; ch = getchar()) if (ch == '-') f = -1;
  for (; ch >= '0' && ch <= '9'; ch = getchar()) x = x * 10 + ch - '0';
  return x * f;
}

const int numlen = 10005; // 位数

struct bign {
  int len, s[numlen];
  bign() {
    memset(s, 0, sizeof(s));
    len = 1;
  }

  bign(int num) {
    *this = num;
  }

  bign(const char *num) {
    *this = num;
  }

  bign operator=(const int num) {
    char s[numlen];
    sprintf(s, "%d", num);
    *this = s;
    return *this;
  }

  bign operator=(const char *num) {
    len = strlen(num);
    while (len > 1 && num[0] == '0') num++, len--;
    for (int i = 0; i < len; i++) s[i] = num[len - i - 1] - '0';
    return *this;
  }

  void deal() {
    while (len > 1 && !s[len - 1]) len--;
  }

  bign operator+(const bign& a) const {
    bign ret;
    ret.len = 0;
    int top = max(len, a.len), add = 0;
    for (int i = 0; add || i < top; i++) {
      int now = add;
      if (i < len) now += s[i];
      if (i < a.len) now += a.s[i];
      ret.s[ret.len++] = now % 10;
      add              = now / 10;
    }
    return ret;
  }

  bign operator-(const bign& a) const {
    bign ret;
    ret.len = 0;
    int cal = 0;
    for (int i = 0; i < len; i++) {
      int now = s[i] - cal;
      if (i < a.len) now -= a.s[i];
      if (now >= 0) cal = 0;
      else {
        cal = 1; now += 10;
      }
      ret.s[ret.len++] = now;
    }
    ret.deal();
    return ret;
  }

  bign operator*(const bign& a) const {
    bign ret;
    ret.len = len + a.len;
    for (int i = 0; i < len; i++) {
      for (int j = 0; j < a.len; j++) ret.s[i + j] += s[i] * a.s[j];
    }
    for (int i = 0; i < ret.len; i++) {
      ret.s[i + 1] += ret.s[i] / 10;
      ret.s[i]     %= 10;
    }
    ret.deal();
    return ret;
  }

  // 乘以小数，直接乘快点
  bign operator*(const int num) {
    bign ret;
    ret.len = 0;
    int bb = 0;
    for (int i = 0; i < len; i++) {
      int now = bb + s[i] * num;
      ret.s[ret.len++] = now % 10;
      bb               = now / 10;
    }
    while (bb) {
      ret.s[ret.len++] = bb % 10;
      bb              /= 10;
    }
    ret.deal();
    return ret;
  }

  bign operator/(const bign& a) const {
    bign ret, cur = 0;
    ret.len = len;
    for (int i = len - 1; i >= 0; i--) {
      cur      = cur * 10;
      cur.s[0] = s[i];
      while (cur >= a) {
        cur -= a;
        ret.s[i]++;
      }
    }
    ret.deal();
    return ret;
  }

  bign operator%(const bign& a) const {
    bign b = *this / a;
    return *this - b * a;
  }

  bign operator+=(const bign& a) {
    *this = *this + a; return *this;
  }

  bign operator-=(const bign& a) {
    *this = *this - a; return *this;
  }

  bign operator*=(const bign& a) {
    *this = *this * a; return *this;
  }

  bign operator/=(const bign& a) {
    *this = *this / a; return *this;
  }

  bign operator%=(const bign& a) {
    *this = *this % a; return *this;
  }

  bool operator<(const bign& a) const {
    if (len != a.len) return len < a.len;
    for (int i = len - 1; i >= 0; i--) if (s[i] != a.s[i]) return s[i] < a.s[i];
    return false;
  }

  bool operator>(const bign& a) const  {
    return a < *this;
  }

  bool operator<=(const bign& a) const {
    return !(*this > a);
  }

  bool operator>=(const bign& a) const {
    return !(*this < a);
  }

  bool operator==(const bign& a) const {
    return !(*this > a || *this < a);
  }

  bool operator!=(const bign& a) const {
    return *this > a || *this < a;
  }

  string str() const {
    string ret = "";
    for (int i = 0; i < len; i++) ret = char(s[i] + '0') + ret;
    return ret;
  }
};
istream& operator>>(istream& in, bign& x) {
  string s;
  in >> s;
  x = s.c_str();
  return in;
}

ostream& operator<<(ostream& out, const bign& x) {
  out << x.str();
  return out;
}

// 大数开平方
bign Sqrt(bign x) {
  int a[numlen / 2];
  int top = 0;
  for (int i = 0; i < x.len; i += 2) {
    if (i == x.len - 1) {
      a[top++] = x.s[i];
    }
    else a[top++] = x.s[i] + x.s[i + 1] * 10;
  }
  bign ret = (int)sqrt((double)a[top - 1]);
  int  xx  = (int)sqrt((double)a[top - 1]);
  bign pre = a[top - 1] - xx * xx;
  bign cc;
  for (int i = top - 2; i >= 0; i--) {
    pre = pre * 100 + a[i];
    cc  = ret * 20;
    for (int j = 9; j >= 0; j--) {
      bign now = (cc + j) * j;
      if (now <= pre) {
        ret  = ret * 10 + j;
        pre -= now;
        break;
      }
    }
  }
  return ret;

  /*
     bign a, b;
     cin >> a >> b;
     a = 1;
     a = "123";
     bign c("123");
     cout << a + b << endl;
     cout << a - b << endl;
     cout << a * b << endl;
     cout << a / b << endl;
     cout << a % b << endl;
     cout << Sqrt(a) << endl;
   */
}
"""
  '矩阵快速幂':
    'prefix': 'fmatrix'
    'body':"""
struct Matrix {
  ll a[maxn][maxn];
  void init() { // 初始化为单位矩阵
    memset(a, 0, sizeof(a));
    for (int i = 0; i < maxn; ++i) {
      a[i][i] = 1;
    }
  }
};

// 矩阵乘法
Matrix mul(Matrix a, Matrix b) {
  Matrix ans;
  for (int i = 0; i < maxn; ++i) {
    for (int j = 0; j < maxn; ++j) {
      ans.a[i][j] = 0;
      for (int k = 0; k < maxn; ++k) {
        ans.a[i][j] += a.a[i][k] * b.a[k][j];
        ans.a[i][j] %= mod;
      }
    }
  }
  return ans;
}

// 矩阵快速幂
Matrix qpow(Matrix a, ll n) {
  Matrix ans;
  ans.init();
  while (n) {
    if (n & 1) ans = mul(ans, a);
    a  = mul(a, a);
    n /= 2;
  }
  return ans;
}

void output(Matrix a) {
  for (int i = 0; i < maxn; ++i) {
    for (int j = 0; j < maxn; ++j) {
      cout << a.a[i][j] << " ";
    }
    cout << endl;
  }
  cout << "-----END-----" << endl;
}
"""
  'template':
    'prefix': '$$$'
    'body':"""

"""

```

{% endfold %}


# Uncrustify
google c++ 代码格式
创建 `myC++.cfg`文件至`.atom/packages/uncrustify/etc/myC++.cfg`
然后把该文件的路径放到`Atom Beautify` 中
{% fold myC++.cfg %}

```c
indent_align_string=true
indent_braces=false
indent_braces_no_func=false
indent_brace_parent=false
indent_namespace=false
indent_extern=false
indent_class=true
indent_class_colon=true
indent_else_if=false
indent_func_call_param=false
indent_func_def_param=false
indent_func_proto_param=false
indent_func_class_param=false
indent_func_ctor_var_param=false
indent_template_param=false
indent_func_param_double=false
indent_relative_single_line_comments=false
indent_col1_comment=true
indent_access_spec_body=false
indent_paren_nl=false
indent_comma_paren=false
indent_bool_paren=false
indent_square_nl=false
indent_preserve_sql=false
indent_align_assign=true
sp_balance_nested_parens=false
align_keep_tabs=false
align_with_tabs=false
align_on_tabstop=false
align_func_params=true
align_same_func_call_params=true
align_var_def_colon=true
align_var_def_attribute=true
align_var_def_inline=true
align_right_cmt_mix=false
align_on_operator=true
align_mix_var_proto=false
align_single_line_func=true
align_single_line_brace=true
align_nl_cont=true
align_left_shift=true
nl_collapse_empty_body=true
nl_assign_leave_one_liners=false
nl_class_leave_one_liners=false
nl_enum_leave_one_liners=false
nl_getset_leave_one_liners=false
nl_func_leave_one_liners=false
nl_if_leave_one_liners=false
nl_multi_line_cond=false
nl_multi_line_define=false
nl_before_case=true
nl_after_case=false
nl_after_return=true
nl_after_semicolon=false
nl_after_brace_open=false
nl_after_brace_open_cmt=false
nl_after_vbrace_open=false
nl_after_brace_close=false
nl_define_macro=true
nl_squeeze_ifdef=false
nl_ds_struct_enum_cmt=true
nl_ds_struct_enum_close_brace=true
nl_create_if_one_liner=true
nl_create_for_one_liner=true
nl_create_while_one_liner=true
ls_for_split_full=false
ls_func_split_full=true
nl_after_multiline_comment=true
eat_blanks_after_open_brace=true
eat_blanks_before_close_brace=true
mod_pawn_semicolon=false
mod_full_paren_if_bool=true
mod_remove_extra_semicolon=true
mod_sort_import=false
mod_sort_using=false
mod_sort_include=false
mod_move_case_break=true
mod_remove_empty_return=true
cmt_indent_multi=true
cmt_c_group=false
cmt_c_nl_start=false
cmt_c_nl_end=false
cmt_cpp_group=false
cmt_cpp_nl_start=false
cmt_cpp_nl_end=false
cmt_cpp_to_c=false
cmt_star_cont=false
cmt_multi_check_last=true
cmt_insert_before_preproc=false
pp_indent_at_level=false
pp_region_indent_code=false
pp_if_indent_code=false
pp_define_at_level=false
indent_columns=2
align_var_def_span=1
align_var_def_star_style=2
align_var_def_amp_style=2
align_var_def_thresh=3
align_var_def_gap=1
align_assign_span=1
align_enum_equ_span=1
align_var_struct_span=1
align_struct_init_span=1
align_typedef_span=1
align_typedef_star_style=2
align_typedef_amp_style=2
align_right_cmt_span=4
align_right_cmt_at_col=1
align_func_proto_span=3
nl_end_of_file_min=1
nl_func_var_def_blk=1
code_width=82
nl_max=3
nl_after_func_proto=0
nl_after_func_body=2
nl_after_func_body_one_liner=2
nl_before_block_comment=2
nl_before_c_comment=2
nl_before_cpp_comment=2
nl_before_access_spec=2
nl_after_access_spec=2
nl_comment_func_def=1
nl_after_try_catch_finally=1
mod_full_brace_nl=1
mod_add_long_ifdef_endif_comment=1
mod_add_long_ifdef_else_comment=1
cmt_width=80
newlines=auto
indent_with_tabs=0
sp_arith=add
sp_assign=add
sp_enum_assign=add
sp_pp_concat=add
sp_pp_stringify=add
sp_bool=add
sp_compare=add
sp_inside_paren=remove
sp_paren_paren=remove
sp_paren_brace=add
sp_before_ptr_star=add
sp_before_unnamed_ptr_star=add
sp_between_ptr_star=remove
sp_after_ptr_star=remove
sp_after_ptr_star_func=add
sp_before_ptr_star_func=remove
sp_before_byref=remove
sp_before_unnamed_byref=remove
sp_after_byref=add
sp_after_byref_func=add
sp_before_byref_func=remove
sp_after_type=add
sp_before_angle=remove
sp_inside_angle=remove
sp_after_angle=remove
sp_angle_paren=remove
sp_angle_word=remove
sp_before_sparen=add
sp_inside_sparen=remove
sp_sparen_brace=add
sp_special_semi=remove
sp_before_semi=remove
sp_before_semi_for=remove
sp_before_semi_for_empty=remove
sp_after_semi_for_empty=remove
sp_before_square=remove
sp_before_squares=remove
sp_inside_square=remove
sp_after_comma=add
sp_before_comma=remove
sp_after_class_colon=add
sp_before_class_colon=add
sp_before_case_colon=remove
sp_after_operator=remove
sp_after_operator_sym=remove
sp_after_cast=remove
sp_inside_paren_cast=remove
sp_cpp_cast_paren=remove
sp_sizeof_paren=remove
sp_inside_braces_enum=add
sp_inside_braces_struct=add
sp_inside_braces=add
sp_inside_braces_empty=remove
sp_type_func=add
sp_func_proto_paren=remove
sp_func_def_paren=remove
sp_inside_fparens=remove
sp_inside_fparen=remove
sp_square_fparen=remove
sp_fparen_brace=add
sp_func_call_paren=remove
sp_func_call_user_paren=remove
sp_func_class_paren=remove
sp_return_paren=add
sp_attribute_paren=remove
sp_defined_paren=remove
sp_throw_paren=remove
sp_macro=add
sp_macro_func=remove
sp_else_brace=add
sp_brace_else=add
sp_brace_typedef=add
sp_catch_brace=add
sp_brace_catch=add
sp_finally_brace=add
sp_brace_finally=add
sp_try_brace=add
sp_getset_brace=add
sp_before_dc=remove
sp_after_dc=remove
sp_not=remove
sp_inv=remove
sp_addr=remove
sp_member=remove
sp_deref=remove
sp_sign=remove
sp_incdec=remove
sp_before_nl_cont=remove
sp_after_oc_scope=remove
sp_after_oc_colon=remove
sp_before_oc_colon=remove
sp_after_send_oc_colon=add
sp_before_send_oc_colon=remove
sp_after_oc_type=remove
sp_cond_colon=add
sp_cond_question=add
sp_cmt_cpp_start=add
nl_end_of_file=add
nl_namespace_brace=remove
nl_class_brace=remove
nl_class_init_args=add
nl_func_decl_args=add
nl_before_if=add
nl_before_for=add
nl_before_while=add
nl_before_switch=add
nl_before_do=add
mod_full_brace_do=remove
mod_full_brace_for=remove
mod_full_brace_if=remove
mod_full_brace_while=remove
mod_paren_on_return=remove
pp_space=add
```

{% endfold %}



