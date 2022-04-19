---
title: "倍数ゼータ・メビウス変換のメモ"
date: 2022-04-19T12:00:00+09:00
tags: ["Algorithm"]
---

[約数ゼータ・メビウス変換]({{< ref "posts/divisor_transform2">}})に続いて、倍数もメモします。

## 倍数ゼータ・メビウス変換

関数$f(n)$に対する倍数ゼータ変換は以下の定義です。$n | m$は「$m$は$n$の倍数」という意味で、$m$はmultipleのmです。

$$
F(n) = \sum\_{ n | m } f(m)
$$


倍数ゼータ変換の逆操作が倍数メビウス変換です。

## 倍数ゼータ・メビウス変換の実装

[高速ゼータ変換の約数版 O(N log(log(N))) - noshi91のメモ](https://noshi91.hatenablog.com/entry/2018/12/27/121649)を参考にすると早い実装が得られます。

```cpp
template <class T>
void multiple_transform(vector<T> &a) {
    int n = a.size();
    vector<bool> sieve(n, true);
    for (int p = 2; p < n; ++p) {
        if (sieve[p]) {
            for (int k = (n - 1) / p; k != 0; --k) {
                sieve[k * p] = false;
                a[k] += a[k * p];
            }
        }
    }
    for (int i = 0; ++i != n;) {
        a[i] += a[0];
    }
}
 
template <class T>
void inverse_multiple_transform(vector<T> &a) {
    int n = a.size();
    vector<bool> sieve(n, true);
    for (int i = 0; ++i != n;) {
        a[i] -= a[0];
    }
    for (int p = 2; p < n; ++p) {
        if (sieve[p]) {
            for (int k = 1; k * p < n; ++k) {
                sieve[k * p] = false;
                a[k] -= a[k * p];
            }
        }
    }
}
```

## gcdとの関連

$f(n)$の定義が「$\gcd$が$n$となる集合についての対応する値の総和」、つまり下のような式の形をしていたとしましょう。

$$
f(n) = \sum\_{n = \gcd(S)} g(S)
$$

これの倍数ゼータ変換$F(n)$を考えると

$$
\begin{align}
F(n) = & \sum\_{n | m} \sum\_{m = \gcd(S)} g(S) \\\\
= & \sum\_{n | \gcd(S)} g(S) \\\\
= & \sum\_{n | S\_0, n | S\_1, \cdots} g(S) \\\\
\end{align}
$$

となって、すべての要素が$n$を約数にもつような集合$S$についての計算になります。  
これがかなりいい性質で、数$A$の約数列挙は$O(\sqrt A)$でできるので、
$F(n)$の$n$それぞれについての計算が高速にできる場合があります。

## AGC038 C - LCMs

[AGC038 C - LCMs](https://atcoder.jp/contests/agc038/tasks/agc038_c)

$$
\begin{align}
& \sum_{i = 0}^{N - 1} \sum\_{j = 0}^{i - 1} \operatorname{lcm}(A\_i, A\_j) \\\\
= & \sum_{i = 0}^{N - 1} \sum_{j = 0}^{i - 1} \frac{A\_i A\_j}{\gcd(A\_i, A\_j)} \\\\
\end{align}
$$

ここで、$g = \gcd(A\_i, A\_j)$となる$i, j \\ (i < j)$それぞれについてまとめて計算することにします。

$$
\begin{align}
= & \sum_{g} \sum\_{g = gcd(A\_i, A\_j), i < j} \frac{A\_i A\_j}{g} \\\\
= & \sum_{g} \frac{1}{g} \sum\_{g = gcd(A\_i, A\_j), i < j} A\_i A\_j \\\\
\end{align}
$$

$f(g) = \sum\_{g = gcd(A\_i, A\_j), i < j} A\_i A\_j$として、これを倍数ゼータ変換をした$F(n)$を考えましょう。

$$
\begin{align}
F(n) & = \sum\_{n | m} f(m) \\\\
& = \sum\_{n | m} \sum\_{m = \gcd(A\_i, A\_j), i < j} A\_i A\_j \\\\
& = \sum\_{n | \gcd(A\_i, A\_j), i < j} A\_i A\_j \\\\
& = \sum\_{n | A\_i, n | A\_j, i < j} A\_i A\_j \\\\
& = \frac{1}{2} ((\sum\_{n | A\_i} A\_i)^2 - \sum\_{n | A\_i} A\_i^2)
\end{align}
$$

$\sum\_{n | A\_i} A\_i$と$\sum\_{n | A\_i} A\_i^2$は、各$i$について$A\_i$の約数を列挙すれば計算できて、全体で$O(N \sqrt A)$で計算できます。あとは計算した$F(n)$を倍数メビウス変換にかければ$f(n)$が得られます。

[提出コード](https://atcoder.jp/contests/agc038/submissions/31088763)

```cpp
constexpr i64 MAX = 1e6 + 1;
 
int main() {
  i64 N;
  cin.tie(nullptr);
  std::ios::sync_with_stdio(false);
  cin >> N;
  vector<i64> A(N);
  rep(i,0,N) cin >> A[i];
  vector<fp> S(MAX);
 
  vector<fp> ans(MAX);
 
  rep(i,0,N) {
    for(i64 d = 1; d * d <= A[i]; d++) {
      if(A[i] % d == 0) {
        S[d] += fp(A[i]);
        ans[d] -= A[i] * A[i];
        if(A[i] / d != d) {
          S[A[i] / d] += fp(A[i]);
          ans[A[i] / d] -= A[i] * A[i];
        }
      }
    }
  }
  fp i2 = fp(2).inv();
  rep(g,1,MAX) {
    ans[g] += S[g] * S[g];
    ans[g] *= i2;
  }
 
  inverse_multiple_transform(ans);
 
  fp res;
  rep(g,1,MAX) {
    res += fp(g).inv() * ans[g];
  }
  cout << res << endl;
}
```

## ちなみに...

$\sum\_{n | A\_i} A\_i$は倍数ゼータ変換の形をしているので、この計算を倍数ゼータ変換して求めることで高速に計算できます。これがいわゆる$\gcd$畳み込みです。

[提出コード](https://atcoder.jp/contests/agc038/submissions/31088850)



```cpp
constexpr i64 MAX = 1e6 + 1;
 
int main() {
  i64 N;
  cin.tie(nullptr);
  std::ios::sync_with_stdio(false);
  cin >> N;
  vector<i64> A(N);
  rep(i,0,N) cin >> A[i];
  vector<fp> S(MAX);
 
  rep(i,0,N) {
    S[A[i]] += A[i];
  }
  multiple_transform(S);
 
  rep(g,1,MAX) {
    S[g] = S[g] * S[g];
  }
 
  inverse_multiple_transform(S);
 
  fp res;
  rep(g,1,MAX) {
    res += fp(g).inv() * S[g];
  }
  rep(i,0,N) {
    res -= A[i];
  }
  cout << res / fp(2) << endl;
}
```

## ちなみに...2

約数を列挙する部分でosa\_k法を用いるとめちゃ速くなります。

[提出コード](https://atcoder.jp/contests/agc038/submissions/31096016)

osa\_kの実装は[えびちゃんのエラトステネスの篩に基づく高速な素因数分解 - Qiita](https://qiita.com/rsk0315_h4x/items/ff3b542a4468679fb409)を参考にしました。

```cpp
struct osa_k {
  using int_type = int;
  std::vector<int_type> min_fact;
 
  // O(NlogN)
  static std::vector<int_type> min_prime_factor(int n) {
    std::vector<int_type> res(n);
    std::iota(std::begin(res), std::end(res), 0);
    for(int i = 2; i * i < n; i++) {
      if(res[i] < i) continue;
      for(int j = i * i; j < n; j += i) {
        if(res[j] == j) res[j] = i;
      }
    }
    return res;
  }
 
  void build(int n) {
    min_fact = min_prime_factor(n);
  }
 
  // O(logN)
  std::vector<std::pair<int_type, int>> prime_factors(int n) const {
    std::vector<std::pair<int_type, int>> res;
    while(n > 1) {
      if(res.empty() || res.back().first != min_fact[n]) {
        res.push_back({ min_fact[n], 0 });
      }
      res.back().second++;
      n /= min_fact[n];
    }
    return res;
  }
  
  // The divisors are not sorted
  // O(logN + |divisors|)
  template<class F>
  void enumerate_divisors(int n, F f) const {
    std::vector<std::pair<int_type, int>> prime_facts = prime_factors(n);
    if(prime_facts.empty()) {
      f(1);
      return;
    }
    std::vector<int> cnt(prime_facts.size());
    std::vector<int> acc(prime_facts.size(), 1);
    while(true){
      f(acc.front());
      int i = 0;
      for(; i < prime_facts.size(); i++) {
        if((cnt[i]++) == prime_facts[i].second) {
          cnt[i] = 0;
        }
        else {
          acc[i] *= prime_facts[i].first;
          break;
        }
      }
      if(i == prime_facts.size()) {
        break;
      }
      while(i --> 0) {
        acc[i] = acc[i + 1];
      }
    }
  }
};

...

int main() {
  i64 N;
  cin.tie(nullptr);
  std::ios::sync_with_stdio(false);
  cin >> N;
  vector<i64> A(N);
  rep(i,0,N) cin >> A[i];
 
  osa_k osa;
  osa.build(MAX);
  vector<fp> S(MAX);
 
  vector<fp> ans(MAX);
 
  rep(i,0,N) {
    osa.enumerate_divisors(A[i], [&](int d) {
        S[d] += fp(A[i]);
        ans[d] -= fp(A[i]) * fp(A[i]);
    });
  }
  fp i2 = fp(2).inv();
  rep(g,1,MAX) {
    ans[g] += S[g] * S[g];
    ans[g] *= i2;
  }
 
  inverse_multiple_transform(ans);
 
  fp res;
  rep(g,1,MAX) {
    res += fp(g).inv() * ans[g];
  }
  cout << res << endl;
}
```


## ABC248 G - GCD cost on the tree

[ABC248 G - GCD cost on the tree](https://atcoder.jp/contests/abc248/tasks/abc248_g)

$P(i, j)$を$(i, j)$パス上の頂点の$A$の値の集合、$d(i, j)$を$(i, j)$パスの長さとすると、

$$
\begin{align}
\sum\_{i < j} (d(i, j) + 1) \gcd(P(i, j))
\end{align}
$$

が答えです。ここから変形していきます。

$$
\begin{align}
= & \sum\_{g} \sum\_{g = \gcd(P(i, j)), i < j} (d(i, j) + 1) g \\\\
= & \sum\_{g} g \sum\_{g = \gcd(P(i, j)), i < j} (d(i, j) + 1) \\\\
\end{align}
$$

$f(g) = \sum\_{g = \gcd(P(i, j)), i < j} (d(i, j) + 1)$として、これの倍数ゼータ変換$F(n)$を考えましょう。

$$
\begin{align}
F(n) = & \sum\_{n | m} \sum\_{m = \gcd(P(i, j)), i < j} (d(i, j) + 1) \\\\
= & \sum\_{n | \gcd(P(i, j)), i < j} (d(i, j) + 1) \\\\
= & \sum\_{P(i, j)の全ての要素はnを約数にもつ, i < j} (d(i, j) + 1)
\end{align}
$$

なので、$A\_i$が$n$を約数にもつような頂点$i$だけからなるグラフを考えると計算できそうです。パスの長さの総和は、各辺を合計何回通るかで計算できます。  
dfs戻りがけを意識して実装すると簡単です。

あとは同様に$F(n)$を倍数メビウス変換をすれば、$f(n)$が求まります。

[提出コード](https://atcoder.jp/contests/abc248/submissions/31087608)

```cpp
i64 N;
vector<i64> A;
vector<vector<int>> G;
vector<vector<int>> dv(1e5 + 1);
 
vector<int> vis;
 
void dfs(int v, int p, int g, vector<i64>& cnt) {
  vis[v] = 1;
  i64 now = 1;
  for(auto u: G[v]) {
    if(u == p) continue;
    if(A[u] % g != 0) continue;
    dfs(u, v, g, cnt);
    now += cnt.back();
  }
  cnt.push_back(now);
}

int main() {
  cin >> N;
  A.resize(N);
  G.resize(N);
  rep(i,0,N) cin >> A[i];
  rep(i,0,N - 1) {
    i64 u, v;
    cin >> u >> v;
    u--;
    v--;
    G[u].push_back(v);
    G[v].push_back(u);
  }
  rep(i,0,N) {
    for(i64 d = 1; d * d <= A[i]; d++) {
      if(A[i] % d == 0) {
        dv[d].push_back(i);
        if(A[i] / d != d) {
          dv[A[i] / d].push_back(i);
        }
      }
    }
  }
  vector<fp> ans(dv.size());
  vis.resize(N, 0);
  rep(g,1,dv.size()) {
    for(auto v: dv[g]){
      if(vis[v] == 0) {
        vector<i64> cnt;
        dfs(v, -1, g, cnt);
        rep(i,0,cnt.size() - 1) {
          ans[g] += fp(cnt[i]) * fp(cnt.back() - cnt[i]);
        }
        ans[g] += fp(cnt.back()) * fp(cnt.back() - 1) / fp(2);
      }
    }
    for(auto v: dv[g]) {
      vis[v] = 0;
    }
  }
  inverse_multiple_transform(ans);
  fp res;
  rep(i,1,dv.size()) {
    res += fp(i) * ans[i];
  }
  cout << res << endl;
}
```


