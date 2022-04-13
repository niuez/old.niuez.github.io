---
title: "グラフ, 格子状のグラフ, 次元拡張グラフを同じコードで扱う抽象化BFS, DijkstraのC++14実装"
date: 2020-04-27T01:08:48+09:00
draft: false
tags: ["Algorithm"]
---

これすごく悩んでいたんですけど, 新しい実装法を考えたらスッキリしたのでまとめます.

先駆者がいたらごめん, 勝手に**にう式グラフ抽象化**って言っちゃおうかな

## 意思

DijkstraやBFSのライブラリを書いておきたい！

## 問題点

- 同じアルゴリズムのライブラリを, 種類の違うグラフに対して一つずつ書くのは本当にイヤ
- 格子状のグラフをわざわざ, `std::vector<std::vector<int>> G`に構築するのは定数倍があるのでイヤ.
- 問題を解くために次元拡張したグラフをわざわざ構築するのは, キレイじゃない
- `struct edge`がC++においてはなんかイヤ, 隣接する頂点が`int to`で固定しないとダメだったりでイヤ(Rustのtraitとかあるならまだしも 個人差あり)
- そもそも`struct edge`を毎回書くのもイヤ(競プロなので)
- 隣接リストを毎回`std::vector<edge>`で返すのもキレイじゃない, できるだけ無駄はなくしたい.

どうしよう(欲求が多すぎる)

## 解決

抽象化しにくいのはなぜかというと, `struct edge`と隣接行列の保持の仕方がグラフの種類ごとに違うからです.  なので, 方針としては

- `struct edge`は, ライブラリに触れさせない
- 隣接行列は, 直接ライブラリに触れさせない

という点を実現できれば良いです. ですが, 隣接行列を`std::vector<なんたら>`で返す関数を返すのも美しくありません. なので, これを関数で受けることにします.

具体的には, 

1. BFS関数は, `delta`という隣接行列を生成する関数を受け取ります.
2. 隣接する頂点を列挙する時は, `delta`に, 現在の頂点$v$と, **$v$と隣接する頂点を引数にとる, 探索をする関数**を渡します.
3. `delta`が隣接する頂点を, 引数で渡された関数の引数に入れて探索をしてもらいます.
4. できる!

Dijkstraの場合には, `delta`に渡す関数を`(V t /* 隣接頂点 */, W cost /* 辺のコスト */)`にしておけばOKですね.

頂点の抽象化は, `index(V) -> int`があればいいです.

## コード

### BFS

```cpp
#include <vector>
#include <queue>
using i64 = long long;

/*
 * delta(V v, fn (V t))
 * index(V v) -> int
 */
template<class V, class Delta, class Index>
std::vector<i64> bfs(std::size_t N, V s, Delta delta, Index index) {
  std::vector<i64> dist(N, -1);
  std::queue<V> que;
  dist[index(s)] = 0;
  que.push(s);
  while(!que.empty()) {
    V v = que.front();
    que.pop();
    delta(v, [&](V t) {
        if(dist[index(t)] == -1) {
          dist[index(t)] = dist[index(v)] + 1;
          que.push(t);
        }
    });
  }
  return dist;
}
```

### Dijkstra

```cpp
#include <vector>
#include <queue>

/*
 * delta(V v, fn (V t, W weight))
 * index(V v) -> int
 */
template<class V, class W, class Delta, class Index>
std::vector<W> dijkstra(std::size_t N, W inf, V s, Delta delta, Index index) {
  std::vector<W> dist(N, inf);
  using P = std::pair<W, V>;
  std::priority_queue<P, std::vector<P>, std::greater<P>> que;
  que.push({ dist[index(s)] = W(), s });
  while(!que.empty()) {
    W d = que.top().first;
    V v = que.top().second;
    que.pop();
    if(dist[index(v)] < d) continue;
    delta(v, [&](V t, W weight) {
        if(dist[index(t)] > dist[index(v)] + weight) {
          que.push({ dist[index(t)] = dist[index(v)] + weight, t });
        }
    });
  }
  return dist;
}
```

### How To Use

格子グラフに関しては, こんな感じに予めライブラリを書いておくと楽です.

```cpp
template<class F>
struct lattice_delta {
  i64 H, W;
  F f;
  using P = std::pair<i64, i64>;
  lattice_delta(i64 H, i64 W, F f): H(H), W(W), f(f) {}
  template<class Func>
  void operator()(P v, Func func) {
    const static vector<i64> dx { 1, 0, -1, 0 };
    i64 i = v.first;
    i64 j = v.second;
    for(i64 q = 0; q < 2; q++) {
      i64 x = i + dx[q];
      i64 y = j + dx[q ^ 1];
      if(0 <= x && x < H && 0 <= y && y < W) {
        f(P(i, j), P(x, y), func);
      }
    }
  }
};
 
template<class F>
lattice_delta<F> make_lattice_delta(i64 H, i64 W, F f) { return lattice_delta<F>(H, W, f); }
 
struct lattice_index {
  i64 H, W;
  using P = std::pair<i64, i64>;
  lattice_index(i64 H, i64 W): H(H), W(W) {}
  i64 operator()(P v) {
    return v.first * W + v.second;
  }
};
```

これで, [Maze Master](https://atcoder.jp/contests/abc151/tasks/abc151_d)は以下のように解くことができます.

```cpp
int main() {
  i64 H, W;
  cin >> H >> W;
  vector<string> S(H);
  rep(i,0,H) {
    cin >> S[i];
  }
  i64 ans = 0;

  using P = std::pair<i64, i64>;
  auto delta = make_lattice_delta(H, W, [&](P v, P t, auto func) {
    if(S[t.first][t.second] == '.') func(t);
    }
  );
  auto index = lattice_index(H, W);

  rep(i,0,H) rep(j,0,W) {
    if(S[i][j] == '#') continue;
    auto res = bfs(H * W, P(i, j), delta, index);
    ans = std::max(ans, *std::max_element(all(res)));
  }
  cout << ans << endl;
}
```

また, 次元拡張グラフを扱う[E - Two Currencies](https://atcoder.jp/contests/abc164/tasks/abc164_e)は, こんな感じで.

```cpp
int main() {
  i64 N, M, S;
  cin >> N >> M >> S;
  S = std::min(S, 2500ll);
 
  struct edge {
    i64 to;
    i64 a;
    i64 b;
  };
 
  vector<vector<edge>> G(N);
  rep(i,0,M) {
    i64 u, v, a, b;
    cin >> u >> v >> a >> b;
    u--;
    v--;
    G[u].push_back({ v, a, b });
    G[v].push_back({ u, a, b });
  }
 
  vector<i64> C(N);
  vector<i64> D(N);
  rep(i,0,N) {
    cin >> C[i] >> D[i];
  }
 
  using P = std::pair<i64, i64>;
  auto index = [&](P p) { return p.first * 2501 + p.second; };
  auto delta = [&](P vv, auto func) {
    i64 v = vv.first;
    i64 c = vv.second;
    func(P(v, std::min(c + C[v], 2500ll)), D[v]);
    for(auto e: G[v]) {
      if(e.a <= c) {
        func(P(e.to, c - e.a), e.b);
      }
    }
  };
  auto res = dijkstra(N * 2501, (i64)(1e17), P(0, S), delta, index);
  rep(i,1,N) {
    i64 ans = res[index(P(i, 0))];
    rep(j,0,2501) {
      ans = std::min(ans, res[index(P(i, j))]);
    }
    cout << ans << "\n";
  }
}
```

C++14ならジェネリックラムダあるので, `auto func`って書き殴れるのでいいですね.

## 提出ページ

- 格子グラフのBFS: [D - Maze Master - 提出 #11773027 - AtCoder Beginner Contest 151](https://atcoder.jp/contests/abc151/submissions/11773027) ちょっとコード違うけど許して
- 格子グラフの01dial(01-bfs): [A - Range Flip Find Route - 提出 #11773947 - AtCoder Grand Contest 043](https://atcoder.jp/contests/agc043/submissions/11773947)
- 次元拡張グラフのDijkstra: [E - Two Currencies - 提出 #12401080 - AtCoder Beginner Contest 164](https://atcoder.jp/contests/abc164/submissions/12401080)
