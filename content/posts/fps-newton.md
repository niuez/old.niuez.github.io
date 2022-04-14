---
title: "FPSのニュートン法メモ"
date: 2022-04-14T15:00:00+09:00
draft: false
tags: ["Algorithm", "FPS"]
---

FPSのライブラリ書き直しに合わせて、[ABC222 H - Beautiful Binary Tree](https://atcoder.jp/contests/abc222/tasks/abc222_h)のニュートン法解(TLE)でニュートン法の勉強をしてみました. [nyaanさんのライブラリ](https://nyaannyaan.github.io/library/fps/formal-power-series.hpp.html)に仕組みが書いてあるのでこれを読みました

$$
G(\hat{g}) \equiv f \pmod{x^n} \\\\
g \equiv \hat{g} - \frac{G(\hat{g}) - f}{G'(\hat{g})} \pmod{x^{2n}}
$$

を使って、$n=1$の初期値から帰納的に計算します。

ABC222Hでは答えの母関数を$A$とすると、

$$
A = x (1 + 3 A + A^2)^2
$$

Aを求めたいので、変形して

$$
G(A) = \frac{A}{(1 + 3 A + A^2)^2} = x = f
$$

$$
G'(A) = \frac{1 - 3A - 3 A^2}{(1 + 3A + A^2)^3} \\\\
A - \frac{G(A) - f}{G'(A)} = A - \frac{(1 + 3A + A^2)(A - x(1 + 3A + A^2)^2)}{1 - 3A - 3A^2}
$$

各段階の計算は$O(n \log n)$なので、全体でも$O(N \log N)$です。この問題だと間に合わないけど、どこかで使えそうだね。

[提出コード](https://atcoder.jp/contests/abc222/submissions/30965301)

```cpp
i64 N;
cin >> N;

fps A({ fp(0) });
fps E({ fp(1) });
E.resize(bound_pow2(N + 1));
int en = E.size();
auto de = std::move(E).dft(E.size());
while(A.size() <= N) {
  i64 n = A.size();
  auto da = A.clone().dft(n * 4);
  fps::DFT da2(vector<fp>(n * 2));
  rep(i,0,n * 2) {
    da2[i] = da[i * 2] * da[i * 2];
  }
  auto db = da2;
  i64 diff = de.size() / db.size();
  rep(i,0,db.size()) {
    db[i] += da[i * 2] * fp(3) + de[i * diff];
  }
  auto B = std::move(db).idft();
  db = std::move(B).dft(n * 4);
  auto B2 = (db * db).idft(2 * n);
  for(int i = B2.size(); i --> 1;) {
    B2[i] = B2[i - 1];
  }
  B2[0] = fp(0);
  B2.resize(4 * n);
  auto C = std::move(db *= (da - std::move(B2).dft(n * 4))).idft(2 * n);
  C.resize(4 * n);
  
  auto dd = da2;
  rep(i,0,dd.size()) {
    dd[i] = dd[i] * fp(3) + da[i * 2] * 3;
  }
  auto D = std::move(dd).idft();
  D[0] -= fp(1);
  auto iD = D.inv(2 * n);
  A.resize(2 * n);
  A += (std::move(C).dft(4 * n) * std::move(iD).dft(4 * n)).idft(2 * n);
}
cout << A[N] << endl;
```

~~にうとんほう~~


