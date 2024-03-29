---
title: "Neovimで電卓"
emoji: "🐶"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Neovim", "Lua", "Calculator"]
published: true
---

この記事は[Vim Advent Calendar 2022(その2)](https://qiita.com/advent-calendar/2022/vim) の 9 日目の記事です。

# Neovimでも電卓使いたいですよね!??

要望に応えて作りました

https://github.com/rapan931/dentaku.nvim/

![dentaku nvim](https://user-images.githubusercontent.com/24415677/205498195-181a9047-cfac-43e2-a80d-d7cd7e6ef331.gif)

## 特徴

-  `hjkl` を使用しフォーカスを移動させ、`<CR>` でキーを押下可能
- 電卓用バッファーにはバッファーローカルなマッピングを追加しているため、カーソルを移動しなくても `1` , `+` , `2` , `=` のような打鍵でキーを押下可能
- Lua の `loadfile()` を使用しているため LuaJit に依存した計算結果を出力
- Windows の電卓アプリの挙動を参考にしている
  - `1 + 2 + 3 = = =` は `1 + 2 + 3 + 3 + 3` となり、計算結果は `12`
  - `1 + 2 + 3 + = =` は `1 + 2 + 3 + (1 + 2 + 3) + (1 + 2 + 3)` となり、計算結果は `18`
  - さぼっているところ(特に符号まわり)もあるため電卓アプリ完全再現はできていません
- `=` キー押下時点の計算式をもとに計算結果を出力
  - `2 + 2 * 3 =` と入力した場合、計算結果は `8` (Windows の電卓アプリでは `12` になる)

## その他

色々やってます

- キー押下時に押されたキーをフラッシュ
- `\%23c` , `%23l` を使ってフォーカス用のハイライト生成
- フォーカスが他の window に移った場合は電卓 window を自動で閉じる
- `scrolloff = 999` を設定して `<C-f>` や `<C-b>` でスクロールしないよう設定
- `<C-o>` で他のファイルに移動しないよう `:clearjumps`
- Neovim リサイズ時に window を中央寄せ
- ヘルプ記載(初めて書いた)
- [vusted](https://github.com/notomo/vusted)を使用したテスト
- カーソルを見えづらくする
  - [fern.nvim](https://github.com/lambdalisue/fern.vim/blob/23dc0773849919cbfcc12f310dd2187e0267c5ed/autoload/fern/internal/viewer/hide_cursor.vim#L8)のコードを参考にしています)
  - (電卓 window 最下行の真ん中にちょっと残ってる)
- CI
  - StyLua を使用したフォーマットチェック
  - vusted を使用したテスト

## 感想

フローティングウィンドウを使ったプラグインを作ってみたかったというのがきっかけです。
楽しく作ることができました。

head の Neovim じゃないと vim.api.nvim_open_win の config に `title` , `title_pos` を指定できないのは CI 落ちて初めて知りました。やはりテスト大事。`<C-^>` で副ファイルに移動しないよう設定したかったけどやり方分からずタイムアップ。また調べます。

時間とやる気があれば、denta9.vim(Vim9script 使って電卓実装)も作ってみたいです

## 最後に

- Happy Vim(Neovim) Life!!
