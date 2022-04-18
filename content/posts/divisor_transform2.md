---
title: "約数ゼータ・メビウス変換のメモ"
date: 2022-04-18T19:00:00+09:00
tags: ["Algorithm"]
---

[以前僕が書いた記事]({{< ref "posts/divisor_transform_memo">}})を復習するときにとても読みにくかったので、もう一度書き直します。

## 約数ゼータ・メビウス変換

関数$f(n)$に対する約数ゼータ変換は以下の定義です。$d|n$は「$d$は$n$の約数」という意味です(dはdivisorのd)。約数の足し合わせですね。

$$
F(n) = \sum\_{ d | n } f(d)
$$


約数メビウス変換は、約数ゼータ変換の逆向きの操作です。つまり、$F(n)$から$f(n)$を求める操作です。

ちなみに、メビウス変換はメビウスの反転公式から[メビウス関数](https://ja.wikipedia.org/wiki/%E3%83%A1%E3%83%93%E3%82%A6%E3%82%B9%E9%96%A2%E6%95%B0)を$\mu(n)$とすれば、

$$
f(n) = \sum\_{d | n} F(d) \mu(\frac{n}{d})
$$

です。メビウス関数については蟻本にも書いてあります。

## 約数ゼータ・メビウス変換の実装

[高速ゼータ変換の約数版 O(N log(log(N))) - noshi91のメモ](https://noshi91.hatenablog.com/entry/2018/12/27/121649)がはやいです！

## gcdの関係

ある$n, m$について、$F(\gcd(n, m))$を考えてみます。$f$は$F$をメビウス変換した関数です。

$$
F(\gcd(n, m)) = \sum\_{d | gcd(n, m)} f(d) = \sum\_{d | n, d | m} f(d)
$$

最後の$\sum$の条件が2つあるので、これをどうにか$d | n$だけにしてみます。

$$
c(d) =
\begin{cases}
1 & d | m \\\\
0 & \text{otherwise}
\end{cases}
$$

とすれば以下のようになります。

$$
F(\gcd(n, m)) = \sum\_{d | n} f(d) c(d)
$$

こうすると例えば、$\sum\_i F(\gcd(n, m\_i))$の計算は

$$
\sum\_i F(\gcd(n, m\_i)) = \sum\_{d | n} f(d) (\sum\_i c\_i(d))
$$

とまとめて計算できます。

## AGC038 C - LCMs

[AGC038 C - LCMs](https://atcoder.jp/contests/agc038/tasks/agc038_c)

$$ 
\begin{align}
& \sum\_{i = 0}^{N - 1} \sum\_{j = 0}^{i - 1} \operatorname{lcm}(A\_i, A\_j) \\\\
= & \sum\_{i = 0}^{N - 1} \sum\_{j = 0}^{i - 1} A\_i \frac{A\_j}{\gcd(A\_i, A\_j)} \\\\
\end{align}
$$

$F(n) = \frac{1}{n}$として、その約数メビウス変換を$f(n)$としましょう。すると、

$$ 
\begin{align}
= & \sum\_{i = 0}^{N - 1} A\_i \sum\_{j = 0}^{i - 1} F(\gcd(A\_i, A\_j)) A\_j \\\\
= & \sum\_{i = 0}^{N - 1} A\_i \sum\_{j = 0}^{i - 1} (\sum\_{d | \gcd(A\_i, A\_j)} f(d) A\_j) \\\\
= & \sum\_{i = 0}^{N - 1} A\_i \sum\_{j = 0}^{i - 1} (\sum\_{d | A\_i, d | A\_j} f(d) A\_j) \\\\
\end{align}
$$

ここで、上の$c(d)$のような係数テクを導入します。

$$
s\_j(d) =
\begin{cases}
A\_j & d | A\_j \\\\
0 & \text{otherwise}
\end{cases}
$$

とすれば、$d | A\_j$の条件を取り除けて

$$ 
\begin{align}
= & \sum\_{i = 0}^{N - 1} A\_i \sum\_{j = 0}^{i - 1} (\sum\_{d | A\_i} f(d) s\_j(d)) \\\\
= & \sum\_{i = 0}^{N - 1} A\_i \sum\_{d | A\_i} f(d) (\sum\_{j = 0}^{i - 1} s\_j(d)) \\\\
\end{align}
$$

$\sum\_{j = 0}^{i - 1} s\_j(d)$は$i$を進めながら更新できます。計算量は、$f$を求める約数メビウス変換が$O(A \log\log A)$、約数の数は$O(\sqrt A)$で抑えられるので、全体で$O(A \log\log A + N \sqrt A)$です。

[提出コード](https://atcoder.jp/contests/agc038/submissions/31070938)

```cpp
const i64 MAX = 1e6 + 1;
 
int main() {
  i64 N;
  cin >> N;
  vector<i64> A(N);
  rep(i,0,N) cin >> A[i];
 
  vector<fp> f(MAX);
  rep(i,1,MAX) {
    f[i] = fp(i).inv();
  }
  inverse_divisor_transform(f);
  
  vector<fp> sum(MAX);
  fp ans;
  for(auto a: A) {
    fp res;
    for(i64 d = 1; d * d <= a; d++) {
      if(a % d == 0) {
        res += f[d] * sum[d];
        sum[d] += fp(a);
        i64 dd = a / d;
        if(dd != d) {
          res += f[dd] * sum[dd];
          sum[dd] += fp(a);
        }
      }
    }
    ans += res * fp(a);
  }
  cout << ans << endl;
}
```

## ABC248 G - GCD cost on the treeのパスグラフ版

[ABC248 G - GCD cost on the tree](https://atcoder.jp/contests/abc248/tasks/abc248_g)のパスグラフ版から考えて、木に応用しようとしたけど失敗しちゃったのでパスグラフの解法だけでもメモっておきます...

$N$要素からなる数列$A\_0, \cdots, A\_{N - 1}$が与えられて以下を求める問題です。

$$
\sum\_{i = 0}^{N - 1} \sum\_{j = 0}^{i - 1} (i - j + 1) \gcd(A\_j, \cdots, A\_i)
$$

式変形していきます。

$$
\begin{align}
& = \sum\_{i = 0}^{N - 1} \sum\_{j = 0}^{i - 1} (i + 1) \gcd(A\_j, \cdots, A\_i) \\\\
& - \sum\_{i = 0}^{N - 1} \sum\_{j = 0}^{i - 1} j \gcd(A\_j, \cdots, A\_i)
\end{align}
$$


$F(n) = n$として、その約数メビウス変換を$f(n)$とします。まずは前半部分を計算しましょう。
$$
\begin{align}
& \sum\_{i = 0}^{N - 1} \sum\_{j = 0}^{i - 1} (i + 1) \gcd(A\_j, \cdots, A\_i) \\\\
= & \sum\_{i = 0}^{N - 1} (i + 1) \sum\_{j = 0}^{i - 1} F(\gcd(A\_j, \cdots, A\_i)) \\\\
= & \sum\_{i = 0}^{N - 1} (i + 1) \sum\_{j = 0}^{i - 1} (\sum\_{d | A\_j, \cdots d | A\_i} f(d)) \\\\
\end{align}
$$

先にあったような係数テクをします。

$$
s\_j(d) = 
\begin{cases}
1 & d | A\_j, \cdots d | A\_{i - 1} \\\\
0 & \text{otherwise}
\end{cases}
$$

とすると、

$$
\begin{align}
= & \sum\_{i = 0}^{N - 1} (i + 1) \sum\_{j = 0}^{i - 1} (\sum\_{d | A\_i} f(d) s\_j(d)) \\\\
= & \sum\_{i = 0}^{N - 1} (i + 1) \sum\_{d | A\_i} f(d) (\sum\_{j = 0}^{i - 1} s\_j(d)) \\\\
\end{align}
$$

$\sum\_{j = 0}^{i - 1} s\_j(d)$は、$i$を進めながら更新できます。$S\_{i}(d) = \sum\_{j = 0}^{i - 1} s\_j(d)$としたとき、

$$
S\_{i + 1}(d) = 
\begin{cases}
1 + S\_i(d) & d | A\_i \\\\
0 & \text{otherwise}
\end{cases}
$$

とすればよいです。実装方法としては、$S\_{i + 1}(d) \neq 0$となる$d$の数が$O(\sqrt A)$で抑えられることを活かす方法と、最後に$S(d)$を更新したときの$i$を記録しておく方法がありそうです。

後半部分も計算します。

$$
\begin{align}
& \sum\_{i = 0}^{N - 1} \sum\_{j = 0}^{i - 1} j \gcd(A\_j, \cdots, A\_i) \\\\
= & \sum\_{i = 0}^{N - 1} \sum\_{j = 0}^{i - 1} j F(\gcd(A\_j, \cdots, A\_i)) \\\\
= & \sum\_{i = 0}^{N - 1} \sum\_{j = 0}^{i - 1} (\sum\_{d | A\_j, \cdots d | A\_i} f(d) j) \\\\
\end{align}
$$

先にあったような係数テクをします。

$$
t\_j(d) = 
\begin{cases}
j & d | A\_j, \cdots d | A\_{i - 1} \\\\
0 & \text{otherwise}
\end{cases}
$$

とすると、

$$
\begin{align}
= & \sum\_{i = 0}^{N - 1} (i + 1) \sum\_{j = 0}^{i - 1} (\sum\_{d | A\_i} f(d) t\_j(d)) \\\\
= & \sum\_{i = 0}^{N - 1} (i + 1) \sum\_{d | A\_i} f(d) (\sum\_{j = 0}^{i - 1} t\_j(d)) \\\\
\end{align}
$$

$T\_{i} = \sum\_{j = 0}^{i - 1} t\_j(d)$も同様に、$i$を進めながら更新できます。

$$
T\_{i + 1}(d) = 
\begin{cases}
j + T\_i(d) & d | A\_i \\\\
0 & \text{otherwise}
\end{cases}
$$

これでパスグラフに関しては解けているはず...？下に検証コードを載せておきます。

```cpp
const i64 MAX = 1e5 + 1;

fp solve(i64 N, vector<i64> A) {
  vector<fp> f(MAX);
  rep(i,0,MAX) f[i] = fp(i);
  inverse_divisor_transform(f);

  fp left;
  {
  vector<pair<int, fp>> sum(MAX, { -1, fp(0) });
  rep(i,0,N) {
    fp res;
    for(i64 d = 1; d * d <= A[i]; d++) {
      if(A[i] % d == 0) {
        auto& [b, s] = sum[d];
        res += (b + 1 == i ? s : fp(0)) * f[d];
        if(b + 1 != i) {
          s = 0;
        }
        b = i;
        s += fp(1);
        if(A[i] / d != d) {
          i64 dd = A[i] / d;
          auto& [b, s] = sum[dd];
          res += (b + 1 == i ? s : fp(0)) * f[dd];
          if(b + 1 != i) {
            s = 0;
          }
          b = i;
          s += fp(1);
        }
      }
    }
    left += res * fp(i + 1);
  }
  }
  {
  vector<pair<int, fp>> sum(MAX, { -1, fp(0) });
  rep(i,0,N) {
    fp res;
    for(i64 d = 1; d * d <= A[i]; d++) {
      if(A[i] % d == 0) {
        auto& [b, s] = sum[d];
        res += (b + 1 == i ? s : fp(0)) * f[d];
        if(b + 1 != i) {
          s = 0;
        }
        b = i;
        s += fp(i);
        if(A[i] / d != d) {
          i64 dd = A[i] / d;
          auto& [b, s] = sum[dd];
          res += (b + 1 == i ? s : fp(0)) * f[dd];
          if(b + 1 != i) {
            s = 0;
          }
          b = i;
          s += fp(i);
        }
      }
    }
    left -= res;
  }
  }
  return left;
}

i64 gcd(i64 a, i64 b) {
  if(b == 0) return a;
  return gcd(b, a % b);
}

fp naive(i64 N, std::vector<i64> A) {
  fp ans;
  rep(i,0,N) {
    i64 G = A[i];
    rep(j,i + 1,N) {
      G = gcd(G, A[j]);
      ans += fp(G) * fp(j - i + 1);
    }
  }
  return ans;
}

int main() {
  rep(s,0,1000) {
    i64 N = 1000;
    mt19937 mt(s);
    uniform_int_distribution<> dist(1, (int)1e5);
    vector<i64> A(N);
    rep(i,0,N) A[i] = dist(mt);
    cout << "seed: " << s << endl;
    //cout << N << endl;
    //cout << A << endl;
    fp so, na;
    cout << "solve: " << (so = solve(N, A)) << endl;
    cout << "naive: " << (na = naive(N, A)) << endl;
    assert(so == na);
  }
}
```

木バージョンが解けないので、がんばる
