---
title: "Tarjan's off-line LCA の実装メモと速度"
date: 2020-02-05T20:33:01+09:00
tags: ["Algorithm"]
---

Tarjan's off-line LCAを書いてみたので, その時のメモです.

**ネタバレ注意**

[No.898 tri-βutree - yukicoder](https://yukicoder.me/problems/no/898) のちょっとしたネタバレが含まれます...

<br>

Tarjan's off-line LCA(lowest common ancestors)は, LCAをoff-lineで$O((N + Q) \alpha (N))$で求めるアルゴリズムです. ($\alpha$は逆アッカーマン関数)

[Tarjan's off-line lowest common ancestors algorithm - Wikipedia](https://en.wikipedia.org/wiki/Tarjan%27s_off-line_lowest_common_ancestors_algorithm)

DFSの帰りがけに, `Union Find`で木の辺を`unite`していく. すると, `Union Find`で表現している集合は, いまたどっている頂点とのLCAが同じになる頂点の集合になります.

![](/images/tarjans_uf_tree.png)

具体例はこんな感じ. いま頂点$6$を見ているとします. 二重線はまだつなげていない辺です.

緑の集合は, 頂点$6$とのLCAが頂点$0$である集合です. また, 青の集合は, 頂点$6$とのLCAが頂点$4$である集合です.  
DFSの戻りってこういうことできるんだなあ

## 実装

注意点はクエリを処理するタイミングで, LCAを求めたい頂点２つのどちらもがdfsされた時であること(なので`ans == -2`を挟んでいる)

```cpp
#include <vector>
#include <iostream>
#include <set>
using namespace std;

struct union_find {
  vector<int> par;
  vector<int> rank;
  union_find(int n) : par(n) , rank(n) {
    for(int i = 0;i < n;i++) par[i] = i;
  }
  int root(int i) {
    return par[i] == i ? i : par[i] = root(par[i]);
  }
  /* unite x, y return parent */
  int unite(int x,int y) {
    x = root(x);
    y = root(y);
    if(x == y) return -1;
    if(rank[x] < rank[y]) {
      par[x] = y;
      return y;
    }
    else {
      par[y] = x;
      if(rank[x] == rank[y]) rank[x]++;
      return x;
    }
  }
};

using i64 = long long;
struct tarjans_offline_lca {
  using E = pair<int, i64>;
  vector<vector<E>> G;
  vector<int> ance;
  union_find uf;
  vector<i64> weight;
  vector<pair<int, int>> query;
  vector<vector<pair<int, int>>> q;
  vector<int> ans;
  tarjans_offline_lca(int n): G(n), ance(n), uf(n), weight(n), q(n) {}

  void add_edge(int a, int b, i64 w) {
    G[a].push_back({ b, w });
    G[b].push_back({ a, w });
  }
  void add_query(int a, int b) {
    int i = query.size();
    query.push_back({ a, b });
    q[a].push_back({ b, i });
    q[b].push_back({ a, i });
  }
  void dfs(int v, int f, i64 W) {
    ance[v] = v;
    weight[v] = W;
    for(auto e: G[v]) {
      int u = e.first;
      i64 w = e.second;
      if(f == u) continue;
      dfs(u, v, w + W);
      uf.unite(u, v);
      ance[uf.root(v)] = v;
    }
    for(auto e: q[v]) {
      int u = e.first;
      int i = e.second;
      if(ans[i] == -1) ans[i] = -2;
      else if(ans[i] == -2) ans[i] = ance[uf.root(u)];
    }
  }
  void offline_lca(int root) {
    ans.assign(query.size(), -1);
    dfs(root, -1, 0);
  }
};
```

使用例です. $N = 10^5$, LCAのクエリ数$Q = 3 * 10^5$で196msならいいんじゃない？ [#426438 No.898 tri-βutree - yukicoder](https://yukicoder.me/submissions/426438)

## HLDが速いんじゃ

HLDなんでこんなに速いんですかね. $O(N + Q \log N)$のはずなんですが... 100ms [#426441 No.898 tri-βutree - yukicoder](https://yukicoder.me/submissions/426441)

## しめ

クエリを二回見てるのがダメなんですかね... LCAやるときはHLDでいいでしょう...  
でも, DFS帰りがけがかなり面白い. どこかで使えるといいな
