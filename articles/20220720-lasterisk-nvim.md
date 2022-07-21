---
title: "Neovim用 luaプラグイン lasterisk.nvimを作った"
emoji: "🐶"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Neovim", "Vim", "lua"]
published: true
---

# lasterisk.nvim

Vimの`*`はデフォルトだと次にヒットする単語にカーソルが移動してしまいますが、[vim-asterisk](https://github.com/haya14busa/vim-asterisk)というプラグインを入れると`*`入力時にカーソルが動かなくなりとても便利です。

私は最近Neovimを使い始めて設定もinit.luaで書いているので使うプラグインもlua製に寄せています。vim-asteriskはVim scriptで書かれたプラグインなので、luaの勉強もかねvim-asteriskのlua実装 [lasterisk.nvim](https://github.com/rapan931/lasterisk.nvim) を作ってみました。

https://github.com/rapan931/lasterisk.nvim

## できること

- カーソルが移動しない normalモードの`*`
- カーソルが移動しない normalモードの`g*`
- 選択領域の検索 (visualモードの`*`)

```lua
vim.keymap.set('n', '*',  function() require("lasterisk").search() end)
vim.keymap.set('n', 'g*', function() require("lasterisk").search({ is_whole = false }) end)
vim.keymap.set('x', 'g*', function() require("lasterisk").search({ is_whole = false }) end)

-- not support visual asterisk & is_whole = true
-- vim.keymap.set('x', '*',  function() require("lasterisk").search() end)
```

vim-asteriskではサポートしてるのに、lasterisk.nvimでは非サポートの機能は[色々あります](https://github.com/rapan931/lasterisk.nvim#differences-from-vim-asterisk)。とりあえず私が満足しているので一旦終了ですが、気が向いたら対応してみたいと思います。

## 実装中の話

- `vim.fn.getpos()`だとbyte単位の col が取れますが、`vim.fn.getcharpos()`だと文字単位の col が取れます。
  `vim.fn.strprt()`も同様にbyte単位に部分文字列を取得する関数ですが、`vim.fn.strcharpart()`では文字単位に部分文字列を取得できます。文字単位便利。
  (ちなみに**Vim9 script**では`'あいうえお'[3:] "=> えお`のように`[:]`が文字単位の指定に変わるようですね。従来のVim scriptだと`'あいうえお'[3:] "=> いうえお`のように3byte目以降の部分文字列取得になります)
  
- `vim.region()`というstart, endを指定するとその選択範囲を返すような便利関数がNeovimに追加されていたのですが、使ってみたところ第4引数の`regtype`に`v`,`V`どちらを指定しても結果が同じになってしまったり、第2, 第3引数に渡した引数が更新されてしまったりと、少し使いづらかったです。結局`vim.region()`は使わずに、`vim.fn.getcharpos()`で頑張りました。

- `vim.region()`を調べているときに見つけたのですが選択範囲文字列を取得するような[プルリク](https://github.com/neovim/neovim/pull/13896)が。いつか使ってみたい。

- 毎回毎回`vim.fn.xxxxx`と書くのがめんどいのでプラグイン用ファイル先頭で`local fn = vim.fn`とするとちょっとだけ楽になります。

- `vim.api.xxxx`はリモートプラグイン用の関数[らしい](https://github.com/willelz/nvim-lua-guide-ja/blob/master/README.ja.md#vim%E5%90%8D%E5%89%8D%E7%A9%BA%E9%96%93)。
  ただのプラグイン内ではあんまり使わない方が良いのかもしれませんが、このあたりの使い分けはよく分かっていないです。

## 終わりに

初めてluaでプラグインを書いてみましたが割とすんなりかけました。`vim.fn.xxxx`にVim scriptの関数がごっそり入っているのでVim scriptに慣れている人はそこまで抵抗なく書けると思います。またネタ見つけて書こう。
