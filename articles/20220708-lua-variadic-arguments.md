---
title: "Luaの可変長引数は式"
emoji: "🐶"
type: "tech"
topics: ["Neovim", "Vim", "Lua", "Ruby"]
published: true
---

Neovim 用の設定を Lua で書いているときに可変長引数で少しはまったので記事にまとめます。

# Luaの可変長引数は式!

Lua の可変長引数は式です。

可変長引数が式ではない例として、まずは Ruby の可変長引数を見て見ましょう。

## Rubyの可変長引数

```Ruby
def print_type(*args)
  puts args.class
end
print_type() #=> Array
print_type(1) #=> Array
print_type(1, 2, "str") #=> Array
print_type("str", 2, 1) #=> Array
print_type({hoge: 1, fuga: 1}) #=> Array
print_type(2, {hoge: 1, fuga: 1}) #=> Array
```

配列ですね。引数にアクセスするときは `args[0]` , `args[1]` のようにアクセスします。

## Vimの可変長引数

Vim はどうでしょうか？
Vim の場合 `a:000` に全可変長引数が格納されていて `a:000[0]` , `a:000[1]` で引数にアクセスできます `a:000` が何かというと、こちらもリストです。

```vim
" Number:     0 (v:t_number)
" String:     1 (v:t_string)
" Funcref:    2 (v:t_func)
" List:       3 (v:t_list)
" Dictionary: 4 (v:t_dict)
" Float:      5 (v:t_float)
" Boolean:    6 (v:true and v:false)
" Null:       7 (v:null)
" Blob:      10 (v:t_blob)
function s:print_type(...) abort
  echo type(a:000)
endfunction
call s:print_type() " => 3
call s:print_type(1) " => 3
call s:print_type(1, 2, "str") " => 3
call s:print_type("str", 1, 2) " => 3
call s:print_type({"hoge": 1, "fuga": 1}) " => 3
call s:print_type(2, {"hoge": 1, "fuga": 1}) " => 3
```

また `a:1` , `a:2` でも可変長引数にアクセスできます。
`a:` は何かというと、こちらは辞書です。

```vim
echo type(a:) #=> 4
echo type(g:) #=> 4
echo type(v:) #=> 4
```

## Luaの可変長引数

では Lua はどうでしょうか？

```lua
local function print_type(...)
  print(type(...))
end

print_type() --> bad argument #1 to 'type' (value expected)
print_type(1) --> number
print_type(1, 2, "str") --> number
print_type("str", 1, 2) --> string
print_type({hoge =  1, fuga =  1}) --> table
print_type(2, {hoge =  1, fuga =  1}) --> number
```
最初の例以外は第一引数の型を示しているように見えます(Lua の `type()` は第二引数以降を無視します)。

引数未指定の場合は、エラーになっています。
最初、引数無しの場合になぜエラーが出るのか分かりませんでした。`...` という引数が存在しているので、中には何かしら入っているはず！と思いこんでいました。引数無しの場合 `...` は nil になる？とも思ったのですが、結果は `"nil"` ではなくエラーになってしまいます。

困ったので、[ドキュメント](https://inzkyk.xyz/lua_5_4/language/expressions/)を見たところ、以下の記載がありました
(私は英語あまり読めないので翻訳記事参照します。日本語訳本当にありがたい..)

> 三つのドット (...) で表される可変長引数式は可変長引数を取る関数の直下でのみ利用できます。

可変長引数 **式** !!

そう、Lua では可変長引数は式だったんです。引数無しの場合、 `...` は戻り値無しの関数( `function f() end` )のようになりますし、引数が複数の場合は、 `...` は戻り値が多値の関数( `function f() return 1,2,"str" end` )のようになります。

つまり、最初の例は以下のような形になります。
```lua
local function print_type(f)
  print(type(f()))
end
print_type(function() return end) -- bad argument error
print_type(function() return 1 end)
print_type(function() return 1, 2, "str" end)
print_type(function() return "str", 1, 2 end)
print_type(function() return {hoge =  1, fuga =  1} end)
print_type(function() return 2, {hoge =  1, fuga =  1}end)
```

## 終わりに

`...` が変数だと思い込んでしまい、何かしらの値が存在すると決め込んでいたのが良くなかったです。`...` は式!!と分かると、理解するのは簡単でした。`...` は何も返さない可能性がある。と認識したうえで可変長引数式を今後使っていきたいです。
