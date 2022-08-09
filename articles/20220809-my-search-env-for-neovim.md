---
title: "カーソル下の検索文字をフラッシュ & 検索件数をecho"
emoji: "🐶"
type: "tech"
topics: ['Lua', 'Neovim']
published: true
---

個人的な趣味でピカピカしているものが好きなので、Windows + gVim 環境では `n`, `N` 打鍵時にカーソル下の検索文字をフラッシュさせてました。あと[vim-anzu](https://github.com/osyo-manga/vim-anzu)を使って検索件数をechoしています(カーソルの残像残っちゃっているのはご愛嬌ということで)

![](https://storage.googleapis.com/zenn-user-upload/6e6360d07fdf-20220809.gif)

Neovimでも同じ挙動にしたかったので、Luaでプラグインを2つ作成しました。
検索時のフラッシュは[utahraptor.nvim](https://github.com/rapan931/utahraptor.nvim)、検索件数の表示は[bistahieversor.nvim](https://github.com/rapan931/bistahieversor.nvim)で行えます

https://github.com/rapan931/utahraptor.nvim

https://github.com/rapan931/bistahieversor.nvim

normalモードの`*`打鍵時も検索件数のecho、フラッシュを行って欲しいので
[lasterisk.nvim](https://github.com/rapan931/lasterisk.nvim)とも組み合わせ、以下設定で一応個人的に満足した設定になってます。

```lua
vim.keymap.set({'n', 'x', 'o'}, 'n', function()
  require('bistahieversor').n_and_echo()
  require('utahraptor').flash()
end)
vim.keymap.set({'n', 'x', 'o'}, 'N', function()
  require('bistahieversor').N_and_echo()
  require('utahraptor').flash()
end)

-- `*` でもフラッシュと検索結果表示
vim.keymap.set('n', '*',  function()
  require('lasterisk').search()
  require('bistahieversor').echo()
  require('utahraptor').flash()
end)

-- /hoge<CR>時にも検索件数表示させたいので、CmdlineLeaveに設定追加
vim.api.autocmd('CmdlineLeave', {
  group = 'vimrc_augroup',
  pattern = {'/', '\\?'},
  callback = function()
    if vim.v.event.abort == false then
      vim.api.nvim_feedkeys(vim.api.nvim_replace_termcodes(':lua require("bistahieversor").echo()<CR>',true,false,true),'n',true)
    end
  end
})
```

設定後のNeovimの挙動がこちら

![](https://storage.googleapis.com/zenn-user-upload/60c20298a3ec-20220809.gif)

満足....

ちなみに検索件数の表示についてですが、実はプラグイン使わなくてもVim標準で既に検索件数を表示する機能が存在します。
`set shortmess-=S` で以下のように検索件数が右下に表示されます。(Neovimではshortmessにデフォルトで`S`が含まれていないので、特に設定しなくくても検索件数が表示されます)

![](https://storage.googleapis.com/zenn-user-upload/1441231a207d-20220809.png)

私はgVimの時からvim-anzuを私用して左下に検索文字&件数を表示していたので、右下での表示は慣れませんでした。

## 開発中の話

- 検索実行直後にも検索件数を表示させたかったのですが、Neovimの場合この[issue](https://github.com/neovim/neovim/issues/19519)で報告しているようにCmdlineLeave後のカーソル位置が扱いづらい場所にあり、`nvim_feedkeys()`を使ってお茶を濁しています

- 検索文字のフラッシュは
  1. matchaddでハイライト
  2. その後良い感じのタイミングでmatchdeleteでハイライト除去

  を行っています。良い感じのタイミングというのが結構難しく以下判定を42ms毎にチェックするようにしました。
  - フラッシュ時間(デフォルト500ms)を超えた場合
  - カーソル位置が変わった
  - カレントウィンドウのWindow IDが変わった
  
  現状上手いこと動いていそうに見えますが、何かエッジケース見つけたらまた対応します。
- `matchdelete()` にWindow IDを指定できることを知らず少しはまりました。
  1. 検索文字フラッシュ
  2. フラッシュしている最中にカレントWindow変更
  3. `matchdelete`が呼び出される
  4. そんなIDないよ！！！ と怒られる

  matchdelete()使うときはWindow IDをちゃんと指定しましょう。あとちゃんとヘルプは読みましょう。
- 検索文字のフラッシュについては今把握できているところでいうと以下は非対応です。他にもまだまだ大量に対応できていないものあると思いますが、対応出来そうなものあれば対応します。
  - `very magic`
    - `/\vhoge`
  - カーソル位置先頭にヒットしないケース
    - `/hoge\zshuga`
    - `/hoge/1`

## 最後に

他にもピカピカさせたい部分あるので、引き続きNeovim環境いじっていきたいと思います
(エディターの環境整えている時間が一番好き....)