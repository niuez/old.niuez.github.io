---
title: "RSQをクエリ平方分割で解いた時のメモ"
date: 2020-04-17T10:19:21+09:00
draft: false
tags: ["Algorithm"]
---

バケットサイズなんもわからん. 誰か助けて

## クエリ平方分割

[Point Add Range Sum - Library Checker](https://judge.yosupo.jp/problem/point_add_range_sum)をクエリ平方分割で解きます.  
クエリ平方分割はその名の通り, クエリを分割しておいて, 各分割されたクエリで必要部分だけを残してほかは圧縮しておくことで, 高速に計算することができるテクです

Range Compositeに対応するなら, $\mathtt{update}$クエリだけはこんな感じに, 1点にしておく必要があります.

![](/images/QuerySqrt/image1.png)

ただし, 今回はRange Sumなので, $\mathtt{update}$クエリを無視して大丈夫です.

![](/images/QuerySqrt/image2.png)

## コード

Range Compositeバージョン(Range Compositeを解いたとは言ってない)

```cpp
#include <bits/stdc++.h>
using namespace std;
using i64 = long long;
#define rep(i,s,e) for(i64 (i) = (s);(i) < (e);(i)++)
#define all(x) x.begin(),x.end()

int main() {
  std::cin.tie(nullptr);
  std::ios::sync_with_stdio(false);
  int N, Q;
  cin >> N >> Q;
  vector<i64> A(N + 1);
  rep(i,0,N) {
    cin >> A[i];
  }
  vector<std::tuple<int, int, int>> B(Q);
  rep(i,0,B.size()) {
    int a, b, c;
    cin >> a >> b >> c;
    B[i] = { a, b, c };
  }

  const int Qsq = 2048;
  const int Qsh = 11;
  int Qsz = (Q + Qsq - 1) / Qsq;
  std::bitset<505050> s;
  std::bitset<505050> t;
  vector<i64> idx(N + 1, 0);
  vector<i64> Comp(Qsq * 2, 0);
  for(int qi = 0; qi < Qsz; qi++) {
    int start = qi << Qsh;
    int end = std::min(Q, (qi + 1) << Qsh);
    s = 0;
    t = 0;
    for(int i = start; i < end; i++) {
      int a, b, c;
      std::tie(a, b, c) = B[i];
      if(a == 0) {
        s.set(b);
        t.set(b);
      }
      else {
        s.set(b);
        s.set(c);
      }
    }
    Comp.assign(Qsq * 2, 0);
    Comp[0] += A[0];
    for(int i = 1;i < N + 1;i++) {
      idx[i] = idx[i - 1];
      if(t[i - 1] || s[i]) {
        idx[i]++;
      }
      Comp[idx[i]] += A[i];
    }

    for(int i = start; i < end; i++) {
      int a, b, c;
      std::tie(a, b, c) = B[i];
      if(a == 0) {
        A[b] += c;
        Comp[idx[b]] += c;
      }
      else {
        i64 sum = 0;
        for(int j = idx[b]; j < idx[c]; j++) {
          sum += Comp[j];
        }
        cout << sum << "\n";
      }
    }
  }
}
```

[提出ページ](https://judge.yosupo.jp/submission/7486)

Range Sumバージョン

```cpp
#include <bits/stdc++.h>
using namespace std;
using i64 = long long;
#define rep(i,s,e) for(i64 (i) = (s);(i) < (e);(i)++)
#define all(x) x.begin(),x.end()

int main() {
  std::cin.tie(nullptr);
  std::ios::sync_with_stdio(false);
  int N, Q;
  cin >> N >> Q;
  vector<i64> A(N + 1);
  rep(i,0,N) {
    cin >> A[i];
  }
  vector<std::tuple<int, int, int>> B(Q);
  rep(i,0,B.size()) {
    int a, b, c;
    cin >> a >> b >> c;
    B[i] = { a, b, c };
  }

  int Qsq = 2048;
  int Qsh = 11;
  int Qsz = (Q + Qsq - 1) / Qsq;
  std::bitset<505050> s;
  vector<i64> idx(N + 1, 0);
  vector<i64> Comp(4096, 0);
  for(int qi = 0; qi < Qsz; qi++) {
    int start = qi << Qsh;
    int end = std::min(Q, (qi + 1) << Qsh);
    s = 0;
    for(int i = start; i < end; i++) {
      int a, b, c;
      std::tie(a, b, c) = B[i];
      if(a == 1) {
        s.set(b);
        s.set(c);
      }
    }
    Comp.assign(4096, 0);
    Comp[0] += A[0];
    for(int i = 1;i < N + 1;i++) {
      idx[i] = idx[i - 1];
      if(s[i]) {
        idx[i]++;
      }
      Comp[idx[i]] += A[i];
    }

    for(int i = start; i < end; i++) {
      int a, b, c;
      std::tie(a, b, c) = B[i];
      if(a == 0) {
        A[b] += c;
        Comp[idx[b]] += c;
      }
      else {
        i64 sum = 0;
        for(int j = idx[b]; j < idx[c]; j++) {
          sum += Comp[j];
        }
        cout << sum << "\n";
      }
    }
  }
}
```

## Range Sum バケットサイズ(本編)

これをメモするためだけに記事を書いたと言っても過言ではない

$N, Q <= 500,000$でバケットサイズを変えた時の速度検証

|バケットサイズ|速度|提出ページ|
|:-|:-|:-|
|256|1880ms|[7470](https://judge.yosupo.jp/submission/7470)|
|512|1000ms|[7476](https://judge.yosupo.jp/submission/7476)|
|1024|627ms|[7477](https://judge.yosupo.jp/submission/7477)|
|2048|489ms|[7479](https://judge.yosupo.jp/submission/7479)|
|4096|500ms|[7480](https://judge.yosupo.jp/submission/7480)|
|8192|615ms|[7492](https://judge.yosupo.jp/submission/7492)|

へ〜(空気)

loop unrollとかすると早くなりました.

## 〆

バケットサイズなんもわからん
