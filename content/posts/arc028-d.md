---
title: "ARC028 D - 注文の多い高橋商店"
date: 2022-04-13T15:00:00+09:00
draft: false
tags: ["Editorial", "FPS"]
---

[ARC028 D - 注文の多い高橋商店](https://atcoder.jp/contests/arc028/tasks/arc028_4)

まず、クエリを無視して計算をします。このときのFPSを$F$とおくと、

$$
\begin{eqnarray}
F &=& \prod\_{i = 1}^{N} (1 + \cdots + x^{a\_i}) \\\\
&=& \prod\_{i = 1}^{N} \frac{1 - x^{a\_i + 1}}{1 - x} \\\\
&=& (1 - x)^{-N} \prod\_{i = 1}^{N} (1 - x^{a\_i + 1})
\end{eqnarray}
$$

[EDPC M - Candies](./edpc-m/)と同じ式となり$O(NM)$で計算できます。

次に、$k$種類目の商品を抜いて計算することを考える。$k$種類目の商品を抜いた時のFPSを$G\_k$とすれば、

$$
\begin{eqnarray}
G\_k &=& F \frac{1 - x}{1 - x^{a\_k + 1}} \\\\
     &=& F (1 + x^{a\_k + 1} + x^{2(a\_k + 1)} + \cdots) (1 - x)
\end{eqnarray}
$$

$1 + x^{a\_k + 1} + x^{2(a\_k + 1)} + \cdots$は周期$a\_k + 1$ごとに累積和を取る操作なので、$G\_k$は$O(K)$で求められます。

クエリ$k, x$に対しては$[x^{M - x}]G\_k$で答えることができて、全体で$O(NK)$で求められます。

[提出コード](https://atcoder.jp/contests/arc028/submissions/30922608)

```cpp
i64 N, K, Q;
cin >> N >> K >> Q;
vector<i64> A(N);
rep(i,0,N) {
  cin >> A[i];
  A[i]++;
}

vector<i64> R(Q), C(Q);
rep(i,0,Q) cin >> R[i] >> C[i];
vector<fp> dp(K + 1);
dp[0] = 1;
rep(i,0,N) {
  for(i64 j = K + 1; j --> A[i];) {
    dp[j] -= dp[j - A[i]];
  }
}
rep(i,0,N) {
  rep(j,1,K + 1) {
    dp[j] += dp[j - 1];
  }
}
vector<vector<pair<i64, i64>>> qry(N);
rep(i,0,Q) {
  R[i]--;
  qry[R[i]].push_back( { C[i], i } );
}

vector<fp> ans(Q);

rep(r,0,N) {
  vector<fp> nxt(K + 1);
  vector<fp> sum(A[r]);
  rep(i,0,K + 1) {
    sum[i % A[r]] += dp[i];
    nxt[i] = sum[i % A[r]];
  }
  for(i64 j = K + 1; j --> 1;) {
    nxt[j] -= nxt[j - 1];
  }
  for(auto [c, q]: qry[r]) {
    ans[q] = nxt[K - c];
  }
}
rep(i,0,Q) {
  cout << ans[i] << "\n";
}
```
