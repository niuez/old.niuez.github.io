---
title: "Library Checker - Associative Array に関するいろいろ"
date: 2020-06-12T23:02:44+09:00
draft: false
tags: ["Algorithm"]
---

[Library Checker - Associative Array]を解くに当たってのいろいろをまとめておきます.

## 概要

Associative Arrayは, Mapとよく呼ばれるデータ構造です. キーとそれに対応する値の組を保持し, キーで検索して値を参照することができます.

Associative Arrayを実現するデータ構造は,

- Map型の平衡二分探索木(ただし, キーが比較可能であることが条件)
  - `std::map`
  - 自作
- HashMap
  - `std::unordered_map`
  - 自作(メインテーマ)


## `std::map`: 523ms

[Submit Info #12076](https://judge.yosupo.jp/submission/12076)

これを基準にしていきます

## `merge/split型 AVL Tree`: 682ms

[Submit Info #12082](https://judge.yosupo.jp/submission/12082)

`insert/erase`の非再帰AVL Treeを持ってません, すみません...  
`merge/split`の非再帰AVL Treeを無理やりMapにしたものです. 余分なデータとか, 探索をしているのでもっと早くなると思います.

## `std::unordered_map`: TLE(?????)

[Submit Info #12083](https://judge.yosupo.jp/submission/12083)

`std::unordered_map`だとTLEします. `unordered_map_killer_01`にやられていますね.  
これは, [std::unordered_mapのhash衝突による速度低下をさせてみる - うさぎ小屋](https://kimiyuki.net/blog/2017/03/08/unordered-map-hash-collision/)で紹介されているように, `std::unordered_map`をそのまま使うとハッシュ衝突をより起こすケースでTLEになってしまいます. 対策としては,

- `std::map`を使う
- `hash`関数をランダムに変える

です. hash関数については, 下に書きます.

## 自作HashMap: 114ms

[Submit Info #9787](https://judge.yosupo.jp/submission/9787)

自作する際には, この[高速なハッシュテーブルを設計する | POSTD](https://postd.cc/designing-a-fast-hash-table/)という記事を参考にしました. とてもわかりやすいので, 自作する, しないにしろ読んでおいて損はないです.

- オープンアドレス法: 衝突したら, テーブルの空いている別の場所を探す
- 2冪の制約: mod操作が早いので
- 線形探索法: 衝突したらインデックスをインクリメントしていって探索する
- ハッシュをメモしない

という方針で実装しました. 

ハッシュ関数については, [Gist - badboy/inthash.md Integer Hash Function](https://gist.github.com/badboy/6267743)を参考にしました. ハッシュ関数の実装と解説が載っています. 解説は読み切れていません...

## `std::unordered_map with Hashu64`: 352ms

[Submit Info #12087](https://judge.yosupo.jp/submission/12087)

ハッシュ関数を変えると`std::unordered_map`でもACできましたが, 自作のほうがめっちゃ早いですね

## しめ

自作HashMapを持っておくと, 他のデータ構造に後少し速さを求めたいときに使うことができたりするので便利です.  
`std::map`でACカウントだけしている人はぜひこの機会に書いてみてはいかがでしょうか.
