---
title: "ABC135 D - Digits Parade"
date: 2022-04-13T14:30:00+09:00
draft: false
tags: ["Editorial", "FPS"]
---

[ABC135 D - Digits Parade](https://atcoder.jp/contests/abc135/tasks/abc135_d)

最初に$S$を反転させておきます。

$$
f\_{i} =
\begin{cases}
x^{10^i a} & (S\_i=a) \\\\
\sum\_{j=0}^{9} x^{10^i j} & (S\_i=?)
\end{cases}
$$

とすれば、$\prod_{i = 1}^{N} f\_i$を$x^{13}=1$としたときの$x^5$の係数が答えになります。毎回周期13で集約すれば、$O(13^2 |S|)$です。

[提出コード](https://atcoder.jp/contests/abc135/submissions/30922848)

```cpp
string S;
cin >> S;
reverse(all(S));
i64 N = S.size();
vector<fp> dp(13);
dp[0] = 1;
i64 ten = 1;
rep(i,0,N) {
  vector<fp> nxt(13);
  if(S[i] == '?') {
    rep(j,0,13) {
      rep(k,0,10) {
        nxt[(j + k * ten) % 13] += dp[j];
      }
    }
  }
  else {
    i64 k = S[i] - '0';
    rep(j,0,13) {
      nxt[(j + k * ten) % 13] += dp[j];
    }
  }
  ten *= 10;
  ten %= 13;
  swap(nxt, dp);
}
cout << dp[5] << endl;
```
