---
title: "OyamaC'20 解説"
date: 2020-03-19T20:10:48+09:00
draft: false
tags: ["Editorial"]
---

小山高専で, mitsuさん原案のコンテスト[OyamaC](https://www.hackerrank.com/oyamac)が行われました. testerを担当したので, 解説を書いておきます.

# fizzbuzz

fizzbuzzは書けますか？`N % 15`から書くとスムーズです.

```cpp
#include <bits/stdc++.h>
using namespace std;
using i64 = long long;
int main() {
  i64 N;
  cin >> N;
  if(N % 15 == 0) cout << "fizzbuzz" << endl;
  else if(N % 3 == 0) cout << "fizz" << endl;
  else if(N % 5 == 0) cout << "buzz" << endl;
  else cout << N << endl;
}
```

# tiktak

秒の単位換算です. `3600, 60`で割ったあまりを計算していけばいいです.

```cpp
#include <bits/stdc++.h>
using namespace std;
using i64 = long long;
int main() {
  i64 N;
  cin >> N;
  cout << N / 3600 << endl;
  N %= 3600;
  cout << N / 60 << endl;
  cout << N % 60 << endl;
}
```

# sumsumsum

みつみつみつみたいですね(は？)  これは, $x, y$を$D$をオーバーしない範囲で考えてあげれば, $z$が決まります.
実は, 原案は$1 \le A, B, C, D \le 10^6$でした. あなたは解けますか？

```cpp
#include <bits/stdc++.h>
using namespace std;
using i64 = long long;
#define rep(i,s,e) for(i64 (i) = (s);(i) < (e);(i)++)
#define all(x) x.begin(),x.end()

int main() {
  i64 N = 3;
  vector<i64> A(N);
  rep(i,0,N) {
    cin >> A[i];
  }
  i64 D;
  cin >> D;

  vector<vector<i64>> dp(N + 1, vector<i64>(D + 1, 0));
  dp[0][0] = 1;
  rep(i,0,N) {
    rep(j,0,D + 1) {
      if(j >= A[i]) {
        dp[i + 1][j] = dp[i][j] + dp[i + 1][j - A[i]];
      }
      else {
        dp[i + 1][j] = dp[i][j];
      }
    }
  }
  cout << dp[N][D] << endl;
}
```

# uniform liner sushi

これは蟻本にも書かれている, 区間スゲジューリング問題ですね. 後ろをなるべく小さくするように取る貪欲で解くことができます.

```cpp
#include <bits/stdc++.h>
using namespace std;
using i64 = long long;
#define rep(i,s,e) for(i64 (i) = (s);(i) < (e);(i)++)
#define all(x) x.begin(),x.end()

int main() {
  i64 N;
  cin >> N;
  std::vector<pair<i64, i64>> vec;
  rep(i,0,N) {
    i64 s, t;
    cin >> s >> t;
    vec.push_back({ s + t, s });
  }

  sort(all(vec));


  vector<i64> ne(1, 0);
  i64 ans = 0;
  rep(i,0,N) {
    if(ne[0] <= vec[i].second) {
      ne[0] = vec[i].first;
      ans++;
    }
  }
  cout << ans << endl;
}
```

# rot and rot

これが最後の砦だと思ってたんですけど, 皆さんどうでしたか？  
僕は行列を用いて移動を表現しました. 


<div>
$$
\left(
  \begin{array}{c}
    S & 1
  \end{array}
\right)
$$
</div>

に,

<div>
$$
\left(
  \begin{array}{c}
    A + 1 & 1 \\
    B & 0
  \end{array}
\right)
$$
</div>

を掛けると, 1ステップです. なので, $N$ステップは, 

<div>
$$
\left(
  \begin{array}{c}
    S & 1
  \end{array}
\right)
\left(
  \begin{array}{c}
    A + 1 & 0 \\
    B & 1
  \end{array}
\right) ^ N
= 
\left(
  \begin{array}{c}
    S & 1
  \end{array}
\right)
\left(
  \begin{array}{c}
    (A + 1) ^ N & 0 \\
    ((A + 1)^N - 1) / ((A + 1) - 1) * B & 1
  \end{array}
\right) ^ N
$$
</div>

です.  $A = 0$に注意が必要です. 行列累乗でも間に合うと思います.

```cpp
#include <bits/stdc++.h>
using namespace std;
using i64 = long long;
#define rep(i,s,e) for(i64 (i) = (s);(i) < (e);(i)++)
#define all(x) x.begin(),x.end()

const i64 MOD = 1e9 + 7;

#include <bits/stdc++.h>
using namespace std;
using i64 = long long;

template<i64 M>
constexpr i64 euinv(i64 val) {
    i64 a = M, b = val;
    i64 x = 0, u = 1;
    while (b) {
        i64 t = a / b;
        swap(a -= t * b, b);
        swap(x -= t * u, u);
    }
    return x < 0 ? x + M : x;
}

template<i64 M>
struct modint {
  i64 a;
  constexpr modint(const i64 x = 0) noexcept: a((x % M + M) % M) {}
  constexpr i64 value() const noexcept { return a; }
  constexpr modint inv() const noexcept { return modint(euinv<M>(a)); }
  constexpr modint pow(i64 r) const noexcept {
    modint ans(1);
    modint aa = *this;
    while(r) {
      if(r & 1) {
        ans *= aa;
      }
      aa *= aa;
      r >>= 1;
    }
    return ans;
  }
  constexpr modint& operator+=(const modint r) noexcept {
    a += r.a;
    if(a >= M) a -= M;
    return *this;
  }
  constexpr modint& operator=(const i64 r) {
    a = (r % M + M) % M;
    return *this;
  }
  constexpr modint& operator-=(const modint r) noexcept {
    a -= r.a;
    if(a < 0) a += M;
    return *this;
  }
  constexpr modint& operator*=(const modint r) noexcept {
    a = a * r.a % M;
    return *this;
  }
  constexpr modint& operator/=(modint r) noexcept {
    i64 ex = M - 2;
    while(ex) {
      if(ex & 1) {
        *this *= r;
      }
      r *= r;
      ex >>= 1;
    }
    return *this;
  }

  constexpr modint operator+(const modint r) const {
    return modint(*this) += r;
  }
  constexpr modint operator-(const modint r) const {
    return modint(*this) -= r;
  }
  constexpr modint operator*(const modint r) const {
    return modint(*this) *= r;
  }
  constexpr modint operator/(const modint r) const {
    return modint(*this) /= r;
  }

  constexpr bool operator!=(const modint r) const {
    return this->value() != r.value();
  }

};

template<const i64 M>
std::ostream& operator<<(std::ostream& os, const modint<M>& m) {
  os << m.value();
  return os;
}

using fp = modint<MOD>;

int main() {
  i64 S, A, B, N;
  cin >> S >> A >> B >> N;
  A++;

  if(A == 1) {
    cout << fp(S) + fp(B) * fp(N) << endl;
  }else {
    /*
     * p = 10^9 + 7, F_pでの演算を考える
     * 
     * [S  1] * [ A + 1  0  ]^N
     *          [   B    1  ]
     *
     * = [S  1] * [ (A + 1)^N                              0 ]
     *            [ ((A + 1)^N - 1) / ((A + 1) - 1) * B    1 ]
     */
    cout << fp(S) * fp(A).pow(N) + (fp(A).pow(N) - fp(1)) / (fp(A) - fp(1)) * fp(B) << endl;
  }

}
```

# uniform liner sushi 2

2なのでふたつです. 基本区間スケジュールでやりますが, なるべく寿司が食べ終わるのが遅い方を優先的に食べさせるようにするといいです.

```cpp
#include <bits/stdc++.h>
using namespace std;
using i64 = long long;
#define rep(i,s,e) for(i64 (i) = (s);(i) < (e);(i)++)
#define all(x) x.begin(),x.end()
int main() {
  i64 N;
  cin >> N;
  std::vector<pair<i64, i64>> vec;
  rep(i,0,N) {
    i64 s, t;
    cin >> s >> t;
    vec.push_back({ s + t, s });
  }

  sort(all(vec));


  vector<i64> ne(2, 0);
  i64 ans = 0;
  rep(i,0,N) {
    if(ne[1] <= vec[i].second) {
      ne[1] = vec[i].first;
      ans++;
    }
    else if(ne[0] <= vec[i].second) {
      ne[0] = vec[i].first;
      std::swap(ne[0], ne[1]);
      ans++;
    }
  }
  cout << ans << endl;
}
```

# storage

ちょうど満杯, なので容量の情報は正確に持っておく必要があります. これを添え字にもつDPを考えるとよいです. また, $K$個以内については, 最小で何個を考えてやって, 最後にそれが$K$個以下かを判定すればよいです.

```cpp
#include <bits/stdc++.h>
using namespace std;
using i64 = long long;
#define rep(i,s,e) for(i64 (i) = (s);(i) < (e);(i)++)
#define all(x) x.begin(),x.end()

template<class T>
static inline std::vector<T> ndvec(size_t&& n, T val) noexcept {
  return std::vector<T>(n, std::forward<T>(val));
}

template<class... Tail>
static inline auto ndvec(size_t&& n, Tail&&... tail) noexcept {
  return std::vector<decltype(ndvec(std::forward<Tail>(tail)...))>(n, ndvec(std::forward<Tail>(tail)...));
}

int main() {
  i64 A, N;
  cin >> A >> N;
  i64 K;
  cin >> K;
  vector<i64> W(N);
  rep(i,0,N) {
    cin >> W[i];
  }

  vector<vector<i64>> dp(N + 1, vector<i64>(A + 1, 1e18));
  dp[0][0] = 0;
  rep(i,0,N) {
    rep(j, 0, A + 1) {
      if(j >= W[i]) {
        dp[i + 1][j] = std::min(dp[i][j], dp[i][j - W[i]] + 1);
      }
      else {
        dp[i + 1][j] = dp[i][j];
      }
    }
  }
  if(dp[N][A] <= K) {
    cout << "YES" << endl;
  }
  else {
    cout << "NO" << endl;
  }
}
```
