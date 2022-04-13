---
title: "EDPC M - CandiesをFPSで"
date: 2022-04-13T14:00:00+09:00
draft: false
tags: ["Editorial", "FPS"]
---

[EDPC M - Candies](https://atcoder.jp/contests/dp/tasks/dp_m)

$$
\begin{align}
& \prod\_{i = 1}^{N} (1 + x + \cdots + x^{a\_i}) \\\\
& = \prod\_{i = 1}^{N} \frac{1 - x^{a\_i + 1}}{1 - x} \\\\
& = (1 - x)^{-N} \prod\_{i = 1}^{N} (1 - x^{a\_i + 1})
\end{align}
$$

$\prod\_{i = 1}^{N} (1 - x^{a\_i + 1})$はdpで計算して$O(N K)$、$(1 - x)^{-N}$はN回累積和の操作に対応するので$O(N K)$です。

[提出コード](https://atcoder.jp/contests/dp/submissions/30922282)

```cpp
i64 N, K;
cin >> N >> K;
vector<i64> A(N);
rep(i,0,N) {
  cin >> A[i];
  A[i]++;
}
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
cout << dp[K] << endl;
```

負の二項定理を使えば、最後のパートを$(N + K)$で計算できそう。

[提出コード](https://atcoder.jp/contests/dp/submissions/30942002)

```cpp
i64 N, K;
cin >> N >> K;
vector<i64> A(N);
rep(i,0,N) {
  cin >> A[i];
  A[i]++;
}
vector<fp> dp(K + 1);
dp[0] = 1;
rep(i,0,N) {
  for(i64 j = K + 1; j --> A[i];) {
    dp[j] -= dp[j - A[i]];
  }
}
fact.build(N + K);
fp ans;
rep(i,0,K + 1) {
  ans += dp[i] * fact.binom(K - i + N - 1, N - 1);
}
cout << ans << endl;
```

