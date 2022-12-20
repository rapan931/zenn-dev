---
title: "Luaのテストツール busted の使い方"
emoji: "🐶"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['Lua', 'Neovim', 'Test']
published: true
---

# bustedとは(vustedとは)

Neovim の Lua プラグインのテストを行える[vusted](https://github.com/notomo/vusted)というツールがあることを最近知りました。

`vusted` は[busted](https://github.com/lunarmodules/busted)という Lua のテストツールを Neovim でも動くようラップしたもので、`vusted` を使用するには `busted` の使い方を調べる必要があります。

https://github.com/lunarmodules/busted

本記事ではこの `busted` の使い方を簡単に説明していきます。`busted` を使ってみたいけど日本語記事がないからよく分からん。という人向けの記事ですね。

より詳しく確認したい人は[こちら](https://lunarmodules.github.io/busted/)の `busted` 公式ドキュメントを確認してください。

## Usage

簡単な使い方を説明します。

```lua
-- init_spec.lua
describe("Test", function()
  describe("numerical", function()
    it("'0' is truthy", function()
      assert.is_truthy(0)
    end)

    -- failed test
    it("'1' equal '0'", function()
      assert.is_equal(1, 0)
    end)
  end)

  describe("non-numerical", function()
    it("nil is falsy", function()
      assert.is_falsy(nil)
    end)

    it("table value is same", function()
      assert.is_same({ value = 'same' }, { value = 'same' })
    end)

    it("object is equal", function()
      local a = { obj = 'same' }
      local b = a
      assert.is_equal(a, b)
    end)
  end)
end)
```

`assert.XXXX` でテストを行い、`it` と `describe` にはどんなテストを行うのか記載します。`describe` はネストして使えるので、テスト対象をグループ分けできます。

基本的に `it` には 1 つの assert のみを記載してください。複数の assert を記載した場合、テストが失敗した際にどの assert で失敗したかが出力結果から分かりづらくなってしまいます。
ちなみに `describe` には `context` というエイリアスが存在します。同じく `it` にも `spec` というエイリアスが存在します。
ではこのテストファイルを `busted` で実行してみましょう。

```c
rapan931@rHost:~/tmp/lua$ busted init_spec.lua  --output=TAP
ok 1 - Test numerical '0' is truthy
not ok 2 - Test numerical '1' equal '0'
# init_spec.lua @ 66
# Failure message: init_spec.lua:67: Expected objects to be equal.
# Passed in:
# (number) 0
# Expected:
# (number) 1
ok 3 - Test non-numerical nil is falsy
ok 4 - Test non-numerical table value is same
ok 5 - Test non-numerical object is equal
1..5
```

1 と 0 がイコールな訳はなくこちらのサンプルは極端すぎますが、`describe`,`it` に何をテストするのか記載しておくと、出力されたメッセージを見て何が成功して何が失敗したか分かりやすくなります。

コマンド実行時に `--output=TAP` を指定して出力フォーマットを変更していますが、`TAP` 以外にも `utfTerminal(これがdefault)` や `json` 等用意されています。出力フォーマットはカスタム可能なので、好みの出力が用意されていない場合は独自の出力に変更できます。

## before_each, after_each, setup, teardown

`describe`,`it` ブロック内で `before_each`,`after_each` を使用し対象ブロック内のテスト前後に実行する処理を設定できます。また、`setup`,`teardown` を呼び出すと対象ブロックの最初と最後に実行する処理を設定できます。

```lua
describe("describe 1", function()
  before_each(function() print("before 1") end)
  after_each(function() print("after 1") end)
  setup(function() print("setup 1") end)
  teardown(function() print("teardown 1") end)

  describe("- 1", function()
    it("it 1", function() assert.is_equal(1, 1) end)
  end)

  describe("- 2", function()
    before_each(function() print("before 2") end)
    after_each(function() print("after 2") end)
    setup(function() print("setup 2") end)
    teardown(function() print("teardown 2") end)

    it("it 2", function() assert.is_equal(1, 1) end)
    it("it 3", function() assert.is_equal(1, 1) end)
  end)

  describe("- 3", function()
    it("it 4", function() assert.is_equal(1, 1) end)
  end)
end)
```

実行結果がこちら
```c
rapan931@rHost:~/tmp/lua$ busted init_spec.lua --output=TAP
setup 1
before 1
ok 1 - describe 1 - 1 it 1
after 1
setup 2
before 1
before 2
ok 2 - describe 1 - 2 it 2
after 2
after 1
before 1
before 2
ok 3 - describe 1 - 2 it 3
after 2
after 1
teardown 2
before 1
ok 4 - describe 1 - 3 it 4
after 1
teardown 1
1..4
```

## tag

`describe`,`it` にタグをつけて実行するテストを絞りこむことができます
```lua
describe("a test #tag", function()
  -- tests go here
end)

describe("a nested block #another", function()
  describe("can have many describes", function()
    -- tests
  end)
end)
```

`busted init_spec.lua -t tag` で `#tag` が付いたブロックのみをテストできます。
逆に `busted init_spec.lua --exclude-tag tag` で `#tag` が付いたブロックのテストを除外できます。

## shuffle

`busted --shuffle` でテストの実行順序をランダムにできます。一部テストのみ記載順通りに実行したい場合は対象ブロック内に `randomize(false)` を記載してください

## assert

assert ですが、色々あります

```lua
describe("assert list", function()
  it("test", function()
    assert.True(true) -- trueはluaの予約語なので先頭大文字
    assert.is_true(true)
    assert.is_truthy(0)

    assert.False(false)
    assert.is_false(false)
    assert.is_falsy(nil)

    assert.are.equal(1, 1)
    assert.is.equal(1, 1) -- areはisのエイリアス。is_equal, are_equalも同じ結果になる

    assert.is_not_true(false)
    assert.are_not_equals(1, "1") -- 否定にはnotを付ける

    local a = { key = "value" }
    local b = a
    assert.is_equal(a, b) -- 同じインスタンスかチェック

    assert.is_same("hogehoge", "hogehoge") -- 同じ値かチェック
    assert.is_same({ name = "hoge" }, { name = "hoge" }) -- tableの場合は再帰的に同じ値かチェックする
  end)
end)
```

独自のチェック関数の追加もできます。
例として文字列の先頭 1 文字目が英字であることを確認する assert を追加してみましょう

```lua
local say = require("say")
local function start_witch_alphabet(_, args)
  if not type(args[1]) == "string" or #args ~= 1 then
    return false
  end
  if args[1]:match([=[^[a-zA-Z]]=]) then
    return true
  end
  return false
end

say:set("assertion.start_witch_alphabet.positive", "Expected string begin with an alphabetic character: %s")
say:set("assertion.start_witch_alphabet.negative", "Expected string not begin with an alphabetic character: %s")
assert:register("assertion", "start_witch_alphabet", start_witch_alphabet, "assertion.start_witch_alphabet.positive", "assertion.start_witch_alphabet.negative")

describe("test", function()
  it("begin alphabetic character", function()
    assert.start_witch_alphabet('h123')
  end)
  it("not begin alphabetic character", function()
    assert.not_start_witch_alphabet('0123')
  end)
end)
```

サクッと独自の assert を追加できます。

ちなみに、busted には language pack が存在し、これを使用すれば `busted` 実行時の出力メッセージを日本語に変更できます。
日本語用ファイルの中身見たのですが、面白いメッセージがいくつか格納されていました。
以下に変更後 `busted --lang=ja init_spec.lua` でテストを実行してみてください。シャレオツなエラーメッセージを見ることができますので、気になる人は試してみてください。

```lua
say:set("assertion.start_witch_alphabet.positive", require('busted.languages.ja').failure_messages[6] .. ": %s") -- busted.language.jaのメッセージを使用
say:set("assertion.start_witch_alphabet.negative", require('busted.languages.ja').failure_messages[7] .. ": %s") -- busted.language.jaのメッセージを使用
assert:register("assertion", "start_witch_alphabet", start_witch_alphabet, "assertion.start_witch_alphabet.positive", "assertion.start_witch_alphabet.negative")

describe("test", function()
  it("begin alphabetic character", function()
    assert.start_witch_alphabet('0123') -- わざと失敗させる
  end)
  it("not begin alphabetic character", function()
    assert.not_start_witch_alphabet('h123') -- わざと失敗させる
  end)
end)
```

## spy, mock, stub

`spy`,`stub` を使用すると関数に対して以下をテストできます
- 関数が呼び出されたか
- 何度関数が呼び出されたか
- 関数呼び出し時に渡された引数の値は期待通りか

`spy` は対象の関数をそのまま呼び出しますが、`stub` の場合実際には関数が呼び出されません。
`stub` のサンプルを記載します。

```lua
describe("stubs", function()
  it("replaces an original function", function()
    local t = {
      greet = function(msg) print(msg) end
    }

    stub(t, "greet")

    t.greet("Hey!") -- DOES NOT print 'Hey!'
    assert.stub(t.greet).was.called_with("Hey!")

    t.greet:revert()  -- reverts the stub
    t.greet("Hey!") -- DOES print 'Hey!'
  end)
end)
```

`mock` は `spy`, `stub` どちらにも切り替えられような関数のようです。ここは実際に関数を呼び出したいが、この先は呼び出したくない。。。というような `spy` から `stub` に切り替えたいようなケースで `mock` を使えるかもしれません(正直なところ `mock` をいつ使うのかよく分かっていない)

## private関数

個人的には public な関数のテストで private な関数も使用されため不要と考えていますが、以下のような形で対応無理やりテストできます。

```lua
local my_module = {}
local private_element = {"this", "is", "private"}

function my_module:display()
  print(string.concat(private_element, " "))
end

-- export locals for test
if _TEST then
  -- setup test alias for private elements using a modified name
  my_module._private_element = private_element
end

return my_module
```

## 終わりに

busted についてはシンプルなテストツールで覚えることが少なく割と簡単に使い始めることが出来ました。是非使ってみてください。

また、冒頭でも触れましたが Neovim の Lua プラグインのテスト用に busted をラップした `vusted` というテストツールも存在します。Lua プラグインのテストを書きたい！という人は作者さんが zenn で vusted に関する記事記載されているので是非こちらも見てみてください。

https://zenn.dev/notomo/articles/neovim-lua-plugin-testing

ではでは、よいテストライフを❤