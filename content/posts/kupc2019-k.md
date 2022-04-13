---
title: "KUPC2019 K - One or AllをFPSで"
date: 2022-04-13T13:00:00+09:00
draft: false
tags: ["Editorial", "FPS"]
---

[KUPC2019 K - One or All](https://atcoder.jp/contests/kupc2019/tasks/kupc2019_k)

$$
\begin{align}
&(x + y + z + x^{-1} + y^{-1} + z^{-1} + xyz + (xyz)^{-1})^n \\\\
& = \frac{(1 + xy)^n (1 + xz)^n (1 + yz)^n}{x^n y^n z^n} \\\\
& = (xyz)^{-n} \sum_{i, j, k} \binom{n}{i} \binom{n}{j} \binom{n}{k} (xy)^i (xz)^j (yz)^k \\\\
& = (xyz)^{-n} \sum_{i, j, k} \binom{n}{i} \binom{n}{j} \binom{n}{k} x^{i+j} y^{i+k} z^{j+k} \\\\
\end{align}
$$

次数が$\mod p$されるので、まず$(1 + xy)^n = \sum_{i} \binom{n}{i} (xy)^i$を$\mod p$で集約します。

$i+j \equiv p-n, i+k \equiv q-n, j+k \equiv r-n \\ \pmod p$となる$i, j, k$を求めれば、$x^p y^q z^r$の係数を求められます。

[提出コード](https://atcoder.jp/contests/kupc2019/submissions/30922085)

```cpp
vector<fp> acc(M);
rep(i,0,N + 1) acc[i % M] += fact.binom(N, i);

fp sum = 0;
rep(i,0,M) {
  i64 j = (p - i + M + N) % M;
  i64 k = (q - i + M + N) % M;
  if((j + k - N % M + M) % M == r) {
    sum += acc[i] * acc[j] * acc[k];
  }
}
cout << sum << endl;
```
