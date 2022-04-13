---
title: "約数畳み込みを使って最大公約数と集合をうまく扱うメモ"
date: 2020-02-03T21:20:02+09:00
tags: ["Algorithm"]
---

移植テストです

書いて置かないと頭に置いておけない気がしたのでメモを残す. 間違ってたらごめん

これについて気になったので

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">メビウス関数とかを導入するとより形式的に約数とかを扱えるようになるのかなあ</p>&mdash; Niuez (@xiuez) <a href="https://twitter.com/xiuez/status/1219811848263852033?ref_src=twsrc%5Etfw">January 22, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## 概要

- 約数畳み込み
- メビウス関数
- メビウスの反転公式(約数畳み込みの逆操作)
- 約数畳み込みと逆約数畳み込みのアルゴリズム $O(A \log{\log A})$
- 最大公約数の扱い
- 集合の扱い
- [AGC038C LCMs](https://atcoder.jp/contests/agc038/tasks/agc038_c)の解き方

ネタバレあるので気をつけてください

## 約数畳み込み

関数$f(n)$に対する約数畳み込みとは, 

$\begin{eqnarray} g(n) = \sum\_{ d | n } f(d) \end{eqnarray}$


あとで解説しますが, 方針としてはこの畳み込んだ後の$g(n)$を問題を解けるように定義してやることでGCDを綺麗に扱うことができます.

## メビウス関数

実際に$g(n)$を定義してみます. 一番有名なのは$g(n) = \delta(n, 1)$です. $\delta(n, 1)$はクロネッカーのデルタです. このとき, 

$\begin{eqnarray} g(n) = \delta(n, 1) = \sum\_{ d | n } f(d) \end{eqnarray}$

を満たす$f(n)$はメビウス関数と呼ばれ, $\mu(n)$と書きます.([メビウス関数 - Wikipedia](https://ja.wikipedia.org/wiki/%E3%83%A1%E3%83%93%E3%82%A6%E3%82%B9%E9%96%A2%E6%95%B0))

## メビウスの反転公式(約数畳み込みの逆操作)

上の式のままだと, $f(n)$を導くのは困難です. ここで登場するのがメビウスの反転公式です. これは, 約数畳み込みの逆操作に当たります.

$$ \begin{eqnarray}
g(n) &=& \sum\_{ d | n } f(d) \\\\
f(n) &=& \sum\_{ d | n } g(d) \mu(\frac{n}{d})
\end{eqnarray} $$

これで$g(n)$を定義してから反転公式を適用することで$f(n)$を導くことができます.

## 約数畳み込みと逆約数畳み込みのアルゴリズム

約数畳み込みとその逆はnoshi91さんが計算量$O(A \log{\log A})$で計算するアルゴリズムの記事を紹介しています. 

[http://noshi91.hatenablog.com/entry/2018/12/27/121649](http://noshi91.hatenablog.com/entry/2018/12/27/121649)


逆約数畳み込みの実装例

```cpp
template <class T>
void inverse_divisor_transform(vector<T> &a) {
    int n = a.size();
    vector<bool> sieve(n, true);
    for (int p = 2; p < n; ++p) {
        if (sieve[p]) {
            for (int k = (n - 1) / p; k > 0; --k) {
                sieve[k * p] = false;
                a[k * p] -= a[k];
            }
        }
    }
}
```

## 最大公約数の扱い

自然数$n, m$に対して, 

$\begin{eqnarray} \sum\_{d | n, d | m} f(d) \end{eqnarray}$

を考えると, 

$$
\begin{eqnarray}
\sum\_{d | n, d | m} f(d) &=& \sum\_{d | \gcd(n, m)} f(d) \\\\
&=& g(\gcd(n, m))
\end{eqnarray}
$$

となり, $\gcd(n, m)$に対する操作ができます. 例えば, $f(n) = \mu(n), g(n) = \delta(n, 1)$とすると, $g(\gcd(n, m))$は,「$n, m$が互いに素であれば$1$, そうでなければ$0$」となり, 互いに素かどうかの判定ができます.

## 集合の扱い

例えば, $c\_m(d) = [d | m$]という関数($d$が$m$を割り切るなら$1$, そうでなければ$0$)を考えると,

$$ \sum\_{d | n, d | m} f(d) = \sum\_{ d | n } f(d) c\_m(d) $$

と変形できます.

これを応用します. 自然数の集合$S$ を考え, $c(d) = \sum_{m \in S} c\_m(d)$とすると, 

$$
\begin{eqnarray}
\sum\_{d | n} f(d) c(d) &=& \sum\_{m \in S}\sum\_{d | n} f(d) c\_m(d) \\\\
&=& \sum\_{m \in S} g(\gcd(n, m))
\end{eqnarray}
$$

となります.

$f(n) = \mu(n), g(n) = \delta(n, 1)$を考えてみると, 「集合$S$の中に$n$と互いに素な要素の数」を計算しています.

## AGC038C LCMsを解く


<blockquote><h4><a href="https://atcoder.jp/contests/agc038/tasks/agc038_c">AGC038 C - LCMs</a></h4></blockquote>

$lcm(x, y) = x (\frac{y}{\gcd(x, y)})$と変形します. 約数畳み込みを使う方針でやると, この$(\frac{y}{\gcd(x, y)})$が最後に来てほしい気持ちになります. $g(n) = \frac{1}{n}$と置くと, 

$$
\begin{eqnarray}
\frac{y}{\gcd(x, y)} &=& y \cdot g(\gcd(x, y)) \\\\
&=& \sum\_{d | gcd(x, y)} f(d) y \\\\
&=& \sum\_{d | x, d | y} f(d) y \\\\
&=& \sum\_{d | x} f(d) s\_y(d)
\end{eqnarray}
$$

ここで$s\_y(d)$を「$d$が$y$を割り切るなら$y$, そうでなければ$0$」としました.  
応用して, 自然数の集合$S$ を考え, $s(d) = \sum_{m \in S} s\_m(d)$とすると, 

$$
\begin{eqnarray}
\sum\_{d | x} f(d) s(d) &=& \sum\_{y \in S}\sum\_{d | x} f(d) s\_y(d) \\\\
&=& \sum\_{y \in S} y \cdot g(\gcd(x, y))
\end{eqnarray}
$$

と計算できて, これに$x$を掛けると「集合$S$の中の各要素と$x$の最大公約数の和」を計算できました.  
計算量は, $O(A \log{\log A} + N \sqrt A)$です.


<a href="https://atcoder.jp/contests/agc038/submissions/9703431">C - LCMs の僕の提出</a>

ソースコード

```cpp
#include <bits/stdc++.h>
using namespace std;
using i64 = long long;
#define rep(i,s,e) for(i64 (i) = (s);(i) < (e);(i)++)

/* modint */
/* IO(niu::fin, niu::fout) */

const i64 MOD = 998244353;
using fp = modint<MOD>;


#include <bits/stdc++.h>
using namespace std;
using i64 = long long;

template <class T>
void inverse_divisor_transform(vector<T> &a) {
    int n = a.size();
    vector<bool> sieve(n, true);
    for (int p = 2; p < n; ++p) {
        if (sieve[p]) {
            for (int k = (n - 1) / p; k > 0; --k) {
                sieve[k * p] = false;
                a[k * p] -= a[k];
            }
        }
    }
}

constexpr i64 A = 1e6 + 1;

int main() {
  std::vector<fp> f(A);
  rep(d,1,A) {
    f[d] = fp(d).pow(MOD - 2);
  }
  inverse_divisor_transform(f);

  i64 N;
  niu::fin >> N;
  vector<fp> sum(A);
  fp ans = 0;
  rep(i,0,N) {
    int x;
    niu::fin >> x;
    fp res = 0;
    for(int d = 1; d * d <= x; d++) {
      if(x % d == 0) {
        res += f[d] * sum[d];
        sum[d] += fp(x);
        if(x / d != d) {
          res += f[x / d] * sum[x / d];
          sum[x / d] += fp(x);
        }
      }
    }
    ans += res * fp(x);
  }
  niu::fout << ans.value() << "\n";
}
```
