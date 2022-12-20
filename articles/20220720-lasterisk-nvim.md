---
title: "Neovim用 luaプラグイン lasterisk.nvimを作った"
emoji: "🐶"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Neovim", "Vim", "lua"]
published: true
---

# lasterisk.nvim

Vim の `*` はデフォルトだと次にヒットする単語にカーソルが移動してしまいますが、[vim-asterisk](https://github.com/haya14busa/vim-asterisk)というプラグインを入れると `*` 入力時にカーソルが動かなくなりとても便利です。

私は最近 Neovim を使い始めて設定も init.lua で書いているので使うプラグインも lua 製に寄せています。vim-asterisk は Vim script で書かれたプラグインなので、lua の勉強もかね vim-asterisk の lua 実装 [lasterisk.nvim](https://github.com/rapan931/lasterisk.nvim) を作ってみました。

https://github.com/rapan931/lasterisk.nvim

## できること

- カーソルが移動しない normal モードの `*`
- カーソルが移動しない normal モードの `g*`
- 選択領域の検索 (visual モードの `*`)

```lua
vim.keymap.set('n', '*',  function() require("lasterisk").search() end)
vim.keymap.set('n', 'g*', function() require("lasterisk").search({ is_whole = false }) end)
vim.keymap.set('x', 'g*', function() require("lasterisk").search({ is_whole = false }) end)

-- not support visual asterisk & is_whole = true
-- vim.keymap.set('x', '*',  function() require("lasterisk").search() end)
```

vim-asterisk ではサポートしてるのに、lasterisk.nvim では非サポートの機能は[色々あります](https://github.com/rapan931/lasterisk.nvim#differences-from-vim-asterisk)。とりあえず私が満足しているので一旦終了ですが、気が向いたら対応してみたいと思います。

## 実装中の話

- `vim.fn.getpos()` だと byte 単位の col が取れますが、`vim.fn.getcharpos()` だと文字単位の col が取れます。
  `vim.fn.strprt()` も同様に byte 単位に部分文字列を取得する関数ですが、`vim.fn.strcharpart()` では文字単位に部分文字列を取得できます。文字単位便利。
  (ちなみに**Vim9 script**では `'あいうえお'[3:] "=> えお` のように `[:]` が文字単位の指定に変わるようですね。従来の Vim script だと `'あいうえお'[3:] "=> いうえお` のように 3byte 目以降の部分文字列取得になります)
  
- `vim.region()` という start, end を指定するとその選択範囲を返すような便利関数が Neovim に追加されていたのですが、使ってみたところ第 4 引数の `regtype` に `v`,`V` どちらを指定しても結果が同じになってしまったり、第 2, 第 3 引数に渡した引数が更新されてしまったりと、少し使いづらかったです。結局 `vim.region()` は使わずに、`vim.fn.getcharpos()` で頑張りました。

- `vim.region()` を調べているときに見つけたのですが選択範囲文字列を取得するような[プルリク](https://github.com/neovim/neovim/pull/13896)が。いつか使ってみたい。

- 毎回毎回 `vim.fn.xxxxx` と書くのがめんどいのでプラグイン用ファイル先頭で `local fn = vim.fn` とするとちょっとだけ楽になります。

- `vim.api.xxxx` はリモートプラグイン用の関数[らしい](https://github.com/willelz/nvim-lua-guide-ja/blob/master/README.ja.md#vim%E5%90%8D%E5%89%8D%E7%A9%BA%E9%96%93)。
  ただのプラグイン内ではあんまり使わない方が良いのかもしれませんが、このあたりの使い分けはよく分かっていないです。

## 終わりに

初めて lua でプラグインを書いてみましたが割とすんなりかけました。`vim.fn.xxxx` に Vim script の関数がごっそり入っているので Vim script に慣れている人はそこまで抵抗なく書けると思います。またネタ見つけて書こう。
