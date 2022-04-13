---
title: "Hugoのテーマを変えました & lyla colorschemeについて"
date: 2020-07-24T00:04:58+09:00
tags: ["other"]
---

<style type="text/css" media="screen">
.pallet h2 {
 border-bottom: initial;
}
</style>

## Hugoテーマ変えました

ま〜〜〜たテーマとか色とかいじくってた. 定期的にくる, この抑えられない衝動.  
白はディスプレイにおいてはかなりの強調色なので, それが背景色とはどういうことやねんとは思っていたので, 思い切って自分が使っているカラースキームをHugoにも導入してやろうと思った感じです.

## カラースキームについて

[niuez/lyla.vim](https://github.com/niuez/lyla.vim)で公開しています

<div class="pallet">

<h2 style="color: #6A6868; background-color: #242828">#242828</h2>

- 背景色

<h2 style="color: #242828; background-color: #6A6868">#6A6868</h2>

- グレー
- ボーダー線を引いたり

<h2 style="color: #242828; background-color: #A4A4A4">#A4A4A4</h2>

- 文章の色
- でもあまり強調しない感じ

<h2 style="color: #242828; background-color: #737FC4">#737FC4</h2>

- 深い青
- 目立ってほしくないけどいてほしい単語に使う
- `using`, `return` とか

<h2 style="color: #242828; background-color: #6F8EB3">#6F8EB3</h2>

- ノーマルな青
- Vimではこれを標準の文字の色にしている

<h2 style="color: #242828; background-color: #82BFF5">#82BFF5</h2>

- 明るい青
- ハイライトしたいものに使う
- 検索結果のハイライトとか

<h2 style="color: #242828; background-color: #D06E3E">#D06E3E</h2>

- 深い橙
- 目立ってほしい単語に使う
- `const`, `if`, `while`, `for`とか, 

<h2 style="color: #242828; background-color: #BD8840">#BD8840</h2>

- ノーマルな橙
- 目立って欲しさNo.2の単語に使う

<h2 style="color: #242828; background-color: #E9A05A">#E9A05A</h2>

- 明るい橙
- 明るすぎて使う場所を見失っている(え？
- `h1`にとりあえず使ってはいる

</div>

Neovimではこんな感じです.

![](/images/lyla/screenshot.png)

「目立つ」とは言っても全体にまとまりを感じるこの感じが好き  でも個人差はありそうです.
