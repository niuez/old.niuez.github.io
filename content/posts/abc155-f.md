---
title: "ABC155 F Perils in ParallelをF_2の行列で解く"
date: 2020-02-22T16:40:51+09:00
draft: false
tags: ["Editorial"]
---

[ABC155-F Perils in Parallel - kotatsugameの日記](https://kotatsugame.hatenablog.com/entry/2020/02/19/031012)を読んだときのメモです.

[AtCoder F - Perils in Parallel](https://atcoder.jp/contests/abc155/tasks/abc155_f)を$\mathbb{F}_2$上の行列として考えて解きます. **以下この問題のネタバレを含みます**

## 問題を簡単にする

- 座標$i \  (0 \le i \le N)$のスイッチの状態が$B_i$
- $i \ (0 \le i \le M)$のコードは$\[L_i, R_i) \ (0 \le L_i, R_i \le N)$の範囲のスイッチのオンオフを切り替える.

とします.

## $L, R$を行列で表示

気持ちとしては,

```
(コードを表した行列) * (コードを切ったか切っていないか) = (爆弾のスイッチを切り替えるか切り替えないか)
```

とすると嬉しい. なので, 

$(N, M)$型行列$A$ * $(M, 1)$型行列$\vec{x}$ $=$ $(N, 1)$型行列$B$ という感じに.

サンプル1で試してみます.

```txt
3 4
5 1
10 1
8 0
1 10
4 5
6 7
8 9
```

<div>
$$
B = \left(
  \begin{array}{c}
    1 \\
    0 \\
    1
  \end{array}
\right)
$$
</div>

<div>
$$
A = \left(
  \begin{array}{ccc}
    1 & 1 & 0 & 0 \\
    1 & 0 & 0 & 0 \\
    1 & 0 & 0 & 1
  \end{array}
\right)
$$
</div>

$A$は, 「どこの爆弾のスイッチを切り替えるか」を縦に並べて, それを横にくっつけた感じです.

答えとなる$\vec{x}$は,

<div>
$$
\vec{x} = \left(
  \begin{array}{c}
    1 \\
    0 \\
    0 \\
    1
  \end{array}
\right)
$$
</div>

となります.

## 求め方

$A \vec{x} = B$を解くわけなんですが, このままだと解きにくいので行列の基本変形をすることで, $A$を下三角行列に変換します. [行列の基本変形](https://ja.wikipedia.org/wiki/%E8%A1%8C%E5%88%97%E3%81%AE%E5%9F%BA%E6%9C%AC%E5%A4%89%E5%BD%A2)

普通, 連立方程式を解く時は行の基本変形を行います. しかし, 掃き出し法は計算に時間がかかります. ここでは, 列の基本変形をすることで計算量が落ちることを使います. この証明は後にします.

$A$に基本行列$P\_1 P\_2 \cdots P\_k$を右から掛けて(列の基本変形なので)$A'$と下三角行列に変形したとします. $A' \vec{x'} = B$を解くのは簡単です. 

$\vec{x'}$から$\vec{x}$を求めることを考えます. 

$$ \begin{eqnarray}
A' \vec{x'} &=& B \\\\
A P\_1 P\_2 \cdots P\_k \vec{x'} &=& A \vec{x} \\\\
P\_1 P\_2 \cdots P\_k \vec{x'} &=& \vec{x} \\\\
\end{eqnarray} $$

となるので, $\vec{x'}$を$P\_k$から行の基本変形していけば$\vec{x}$が求まります.

## 列の基本変形による計算量削減

縦方向には, $1$が連続していることを使えば, 計算量が削減できます.

同じ始点$L$を持つ区間, $\[L, R\_1), \[L, R\_2), \cdots, \[L, R\_n)$を$\[L, R\_1), \[R\_1, R\_2), \cdots, \[R\_{n-1}, R\_n)$と掃き出します. すると, 以下のように計算量が計算できます.

掃き出したときに区間が半分未満(ちょうど半分になっていく)になるとすると, $O(\log N)$回しか処理されません. また, スイッチは$M$個なので$O(M \log N)$です.  
区間が半分以上になるのは各$L \ (0 \le L < N)$について$O(\log N)$個しか無いので$O(N \log N)$です.  
合わせて$O((N+M) \log N)$です.  
吐き出す前にソートが必要なので$O((N + M) \log (N) \log ((N + M) \log N))$です.

## コード

[提出 #10248008 - AtCoder Beginner Contest 155](https://atcoder.jp/contests/abc155/submissions/10248008)

```cpp
#include <bits/stdc++.h>
using namespace std;
using i64 = long long;
#define rep(i,s,e) for(i64 (i) = (s);(i) < (e);(i)++)
#define all(x) x.begin(),x.end()


int main() {
  i64 N, M;
  cin >> N >> M;
  vector<pair<i64, i64>> vec;
  rep(i,0,N) {
    i64 a, b;
    cin >> a >> b;
    vec.push_back({ a, b });
  }
  vector<i64> A(N), B(N);
  sort(all(vec));
  rep(i,0,N) {
    A[i] = vec[i].first;
    B[i] = vec[i].second;
  }
  vector<i64> L(M), R(M);
  rep(i,0,M) {
    i64 l, r;
    cin >> l >> r;
    L[i] = lower_bound(all(A), l) - begin(A);
    R[i] = upper_bound(all(A), r) - begin(A);
  }
  vector<vector<pair<i64, i64>>> mat(N);
  rep(i,0,M) {
    if(L[i] < R[i]) {
      mat[L[i]].push_back({ R[i], i });
    }
  }
  vector<pair<i64, i64>> P;
  rep(i,0,N) {
    if(mat[i].size() == 0) continue;
    sort(all(mat[i]));
    for(i64 j = mat[i].size(); j --> 1;) {
      int nl = mat[i][j - 1].first;
      int nr = mat[i][j].first;
      if(nl < nr) {
        mat[nl].push_back({ nr, mat[i][j].second });
        P.push_back({ mat[i][j - 1].second, mat[i][j].second });
      }
    }
  }
  reverse(all(P));
  vector<i64> sum(N + 1, 0);
  vector<i64> ans(M);
  rep(i,0,N) {
    if((B[i] + sum[i]) % 2 == 1) {
      if(mat[i].size() == 0) {
        cout << -1 << endl;
        return 0;
      }
      i64 r = mat[i].front().first;
      i64 idx = mat[i].front().second;
      ans[idx] = true;
      sum[i]++;
      sum[r]--;
    }
    sum[i + 1] += sum[i];
  }
  for(auto p: P) {
    ans[p.first] ^= ans[p.second];
  }
  vector<i64> res;
  rep(i,0,M) {
    if(ans[i]) res.push_back(i);
  }
  cout << res.size() << endl;
  rep(i,0,res.size()) {
    cout << res[i] + 1 << " \n"[i + 1 == res.size()];
  }
}
```

## 〆
