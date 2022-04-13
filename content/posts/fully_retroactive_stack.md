---
title: "Fully Retroactive Stack 実装してみた"
date: 2020-04-10T20:09:09+09:00
draft: false
tags: ["Algorithm"]
---

noshi91さんがFully Retroactive Stackについての解説生放送をやっていたので, その解説通りに実装してみました. ちょっとした解説も書きました.

## できること

Stackの処理できるクエリは以下の通りです.

- $\mathtt{push}(x)$: 要素 $x$ を列の後ろに追加する.
- $\mathtt{pop}()$: 列の一番後ろの要素を取り出す.
- $\mathtt{top}()$: 列の後ろの要素を求める.

これをFully Retroactive Stackに進化させると, こんな感じ.

- $\mathtt{insert}(t, \mathtt{Query})$: 時刻$t$に$\mathtt{Query}$があったとにする.(過去改変)
- $\mathtt{delete}(t)$: 時刻$t$のクエリを無かったことにする.(過去改変)
- $\mathtt{top}(t)$: 時刻$t$時点で$\mathtt{top}()$をしたときの値を求める.

これを$O(\log N)$($N$は$\mathtt{Query}$数)でします.

実は, 今回実装するFully Retroactive Stackは, Stack Arrayとして処理できます. つまり, $\mathtt{top}$よりも強い$\mathtt{at}$が処理できます. これは後ほど.

## アルゴリズム

$\mathtt{push}$と$\mathtt{pop}$をしたときの時系列とStackのサイズをグラフにすると, 例えばこのようになります.

![](/images/FR_Stack/graph1.png)

`A`の時点では, Stackは`[1, 2]`の状態, `B`の時点では, Stackは`[1, 4]`の状態です. こんな感じ.

![](/images/FR_Stack/graph2.png)
![](/images/FR_Stack/graph3.png)

こうみると, 最後に高さ$i$を通過した$\mathtt{push}$が, Stackの$i$番目の要素となります. これを探索できれば勝ちです.

これは, 二分探索で探すことができます. 実際に, `B`の時点での$\mathtt{top}$つまり, 高さ$2$の$\mathtt{push}$を探すことにします.

まず真ん中で区切り, 左右の**最低高度**と**最高高度**を調べます. もし右側の最低高度と最高高度の区間に, 求めたい高さが存在すれば右側を探索します. 逆に存在しなければ, 左側を探索します.

![](/images/FR_Stack/search1.png)

右側が含んでいるので, 右に進みます.

![](/images/FR_Stack/search2.png)

右側が含んでいないので, 左に進みます.

![](/images/FR_Stack/search3.png)

右側が含んでいるので, 右に進みます. すると, $\mathtt{push}(4)$にたどり着きます. これが, `B`の時点での$\mathtt{top}$です. これを考えると, $\mathtt{at}$も処理できそうですね.

`A`の時点での$\mathtt{top}$を調べたいなら, `A`までの区間で探索を開始すればよいです.

## 実装に必要なデータ構造

つまり, 

- 操作の挿入, 削除をする.
- 操作を`split`, `merge`したい.
- 二分探索がしたい.

ので, 操作(グラフでいう, 矢印)を葉に持つ平衡二分探索木を書けば良さそうです.

## 罠

何も入っていないStackを$\mathtt{pop}$すると,`None`を返すような仕様の場合, 上の方法のままだとうまく探索できません. 本来は, サイズが$0$から$0$に遷移するが, グラフのままだと, サイズが$0$から$-1$になってしまいます.

これは, 探索を行う前に最低高度まで潜っておくことで解決できます. こんな感じ.

<blockquote class="twitter-tweet" data-partner="tweetdeck"><p lang="und" dir="ltr"><a href="https://t.co/7YoTD6hftP">pic.twitter.com/7YoTD6hftP</a></p>&mdash; Niuez (@xiuez) <a href="https://twitter.com/xiuez/status/1248253868384768000?ref_src=twsrc%5Etfw">April 9, 2020</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## 実装

親ありmerge/split型AVL葉木です...

```cpp
#include <cstdint>
#include <algorithm>
#include <iostream>
#include <cassert>

struct retroactive_stack {
  using value_type = long long;
  using size_type = std::size_t;
  using height_type = std::int_least32_t;
  using node_index = std::int_least32_t;

  struct diff {
    value_type p_value;
    height_type d;
    height_type max_d;
    height_type min_d;

    static diff push_diff(value_type value) {
      diff i;
      i.p_value = value;
      i.d = 1;
      i.max_d = 1;
      i.min_d = 0;
      return i;
    }
    static diff pop_diff() {
      diff i;
      i.d = -1;
      i.max_d = 0;
      i.min_d = -1;
      return i;
    }

    static diff ope(const diff& l, const diff& r) {
      diff i;
      i.d = l.d + r.d;
      i.max_d = std::max(l.max_d, l.d + r.max_d);
      i.min_d = std::min(l.min_d, l.d + r.min_d);
      return i;
    }
  };

  struct node;

  static struct node n[2020202];
  static size_type ni;

  struct node {
    diff d;
    size_type s;
    height_type h;
    node_index c[2];
    node_index p;
    node() {}
    node& operator[](size_type d) { return n[c[d]]; }
  };

  static node_index new_node(const diff& d) {
    node_index i = ni++;
    n[i].s = 1;
    n[i].h = 1;
    n[i].d = d;
    return i;
  }
  static void fix(node_index i) {
    n[i].s = n[i][0].s + n[i][1].s;
    n[i].h = std::max(n[i][0].h, n[i][1].h) + 1;
    n[i].d = diff::ope(n[i][0].d, n[i][1].d);
  }

  static size_type child_dir(node_index i) {
    if(n[i].p) {
      if(n[n[i].p].c[0] == i) { return 0; }
      else if(n[n[i].p].c[1] == i) { return 1; }
      assert(false);
    }
    return -1;
  }
  static node_index rotate(node_index x, size_type dir) {
    node_index p = n[x].p;
    size_type x_dir = child_dir(x);
    node_index y = n[x].c[1 ^ dir];
    n[n[y][dir].p = x].c[1 ^ dir] = n[y].c[dir];
    n[n[x].p = y].c[dir] = x;
    n[y].p = p;
    if(p > 0) n[p].c[x_dir] = y;
    fix(x);
    fix(y);
    return y;
  }

  static node_index balance(node_index i) {
    fix(i);
    if(n[i][0].h - n[i][1].h == 2) {
      if(n[i][0][0].h - n[i][0][1].h == -1) {
        rotate(n[i].c[0], 0);
      }
      return rotate(i, 1);
    }
    else if(n[i][0].h - n[i][1].h == -2) {
      if(n[i][1][0].h - n[i][1][1].h == 1) {
        rotate(n[i].c[1], 1);
      }
      return rotate(i, 0);
    }
    return i;
  }


  static node_index merge_dir(node_index l, node_index root, node_index r, size_type dir) {
    while(std::abs(n[l].h - n[r].h) > 1) {
      l = n[l].c[dir];
    }
    node_index x = n[l].p;
    n[n[l].p = root].c[dir ^ 1] = l;
    n[n[r].p = root].c[dir] = r;
    fix(root);
    n[n[root].p = x].c[dir] = root;
    x = root;
    while(n[x].p > 0) {
      x = n[x].p;
      x = balance(x);
    }
    return x;
  }

  static node_index merge(node_index l, node_index r) {
    if(!l) return r;
    else if(!r) return l;
    else if(n[l].h >= n[r].h) {
      return merge_dir(l, new_node(diff()), r, 1);
    }
    else {
      return merge_dir(r, new_node(diff()), l, 0);
    }
  }

  static std::pair<node_index, node_index> split(node_index root, size_type pos) {
    if(pos == 0) return { 0, root };
    if(pos == n[root].s) return { root, 0 };
    node_index i = root;
    node_index par = -1;
    while(i > 0 && pos != n[i][0].s) {
      if(pos < n[i][0].s) {
        i = n[i].c[0];
      }
      else {
        pos -= n[i][0].s;
        i = n[i].c[1];
      }
    }
    node_index l = n[i].c[0];
    n[l].p = 0;
    node_index r = n[i].c[1];
    n[r].p = 0;
    size_type dir;
    node_index p = n[i].p;
    node_index pd = child_dir(i);
    while(dir = pd, i = p, i > 0) {
      //std::cout << i << " " << dir << std::endl;
      pd = child_dir(i);
      p = n[i].p;
      n[i].p = 0;
      if(dir == 0) {
        n[i][1].p = 0;
        //std::cout << "merge_dir, 0" << std::endl;
        //debug(n[i].c[1], "");
        //debug(r, "");
        r = merge_dir(n[i].c[1], i, r, 0);
        //debug(r, "");
      }
      else {
        n[i][0].p = 0;
        //std::cout << "merge_dir, 0" << std::endl;
        //debug(n[i].c[0], "");
        //debug(l, "");
        l = merge_dir(n[i].c[0], i, l, 1);
        //debug(l, "");
      }
    }
    return { l, r };
  }

  static node_index at(node_index i, size_type pos) {
    while(n[i].c[0]) {
      if(pos < n[i][0].s) {
        i = n[i].c[0];
      }
      else {
        pos -= n[i][0].s;
        i = n[i].c[1];
      }
    }
    return i;
  }

  static void set(node_index i, size_type pos, diff v) {
    node_index par = -1;
    while(n[i].c[0]) {
      if(pos < n[i][0].s) {
        i = n[i].c[0];
      }
      else {
        pos -= n[i][0].s;
        i = n[i].c[1];
      }
    }
    n[i].d = v;

    while(i = n[i].p, i > 0) {
      fix(i);
    }
  }

  static size_type index(node_index i) {
    size_type pos = 0;
    while(n[i].p > 0) {
      if(child_dir(i) == 1) {
        pos += n[n[i].p][0].s;
      }
      i = n[i].p;
    }
    return pos;
  }

  static node_index search(node_index i, height_type pos) {
    while(n[i].c[0]) {
      if(n[i][0].d.d + n[i][1].d.min_d <= pos && pos < n[i][0].d.d + n[i][1].d.max_d) {
        pos -= n[i][0].d.d;
        i = n[i].c[1];
      }
      else {
        i = n[i].c[0];
      }
    }
    return i;
  }

  static std::pair<node_index, node_index> min_depth_split(node_index i) {
    node_index r = i;
    size_type pos = n[i].d.min_d;
    size_type res = 0;
    while(n[i].c[0]) {
      if(n[i][0].d.d + n[i][1].d.min_d <= pos && pos < n[i][0].d.d + n[i][1].d.max_d) {
        pos -= n[i][0].d.d;
        res += n[i][0].s;
        i = n[i].c[1];
      }
      else {
        i = n[i].c[0];
      }
    }
    return split(r, res);
  }

  static void debug(node_index i, std::string s) {
    if(i == 0) {
      std::cout << s << 0 << std::endl;
      return;
    }
    std::cout << s << i << " = " << n[i].d.p_value << " = " << n[i].p << " = " << n[i].s << std::endl;
    if(n[i].c[0] && n[i][0].p != i) {
      assert(false);
    }
    debug(n[i].c[0], s + "  ");
    if(n[i].c[1] && n[i][1].p != i) {
      assert(false);
    }
    debug(n[i].c[1], s + "  ");
  }


public:

  node_index root;
  retroactive_stack(): root(0) {}
  retroactive_stack(node_index i): root(i) {}

  static node_index new_push_operation(value_type val) {
    return new_node(diff::push_diff(val));
  }
  static node_index new_pop_operation() {
    return new_node(diff::pop_diff());
  }

  retroactive_stack& merge(retroactive_stack right) {
    root = merge(this->root, right.root);
    return *this;
  }
  retroactive_stack split(node_index i) {
    auto p = split(root, index(i));
    retroactive_stack avl;
    avl.root = p.second;
    root = p.first;
    return avl;
  }

  void update(node_index i, diff v) {
    set(root, index(i), v);
  }

  size_type operation_size() {
    return n[root].s;
  }

  height_type stack_size() {
    return n[root].d.d;
  }

  value_type top() {
    auto P = min_depth_split(root);
    auto res = n[search(P.second, n[P.second].d.d - 1)].d.p_value;
    root = merge(P.first, P.second);
    return res;
  }

  void debug() {
    debug(root, "");
  }
};

retroactive_stack::size_type retroactive_stack::ni = 1;
retroactive_stack::node retroactive_stack::n[2020202];

int main() {
  std::vector<int> opes;
  opes.push_back(retroactive_stack::new_push_operation(1));
  opes.push_back(retroactive_stack::new_push_operation(2));
  opes.push_back(retroactive_stack::new_push_operation(3));

  retroactive_stack st;

  st.merge(retroactive_stack::new_pop_operation());
  st.merge(retroactive_stack::new_pop_operation());
  st.merge(retroactive_stack::new_pop_operation());

  for(auto i: opes) {
    st.merge(retroactive_stack(i));
  }
  std::cout << st.top() << std::endl; // 3

  int pop_ope = retroactive_stack::new_pop_operation();

  st.merge(retroactive_stack(pop_ope));

  std::cout << st.top() << std::endl; // 2

  st.merge(retroactive_stack(retroactive_stack::new_pop_operation()));
  std::cout << st.top() << std::endl; // 1

  st.update(pop_ope, retroactive_stack::diff::push_diff(4));
  std::cout << st.top() << std::endl; // 3 (push 1 -> push 2 -> push 3 -> push 4 -> pop)

  auto after = st.split(opes[2]);
  std::cout << st.top() << std::endl; // 2 st := (push 1 -> push 2)
}

```

star押してね(承認欲求)
