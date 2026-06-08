---
title: "明示的オブジェクト引数"
---

基本編のオブジェクト引数の章で、メンバ関数は `.` の左のオブジェクトをオブジェクト引数として受け取ると見た。このオブジェクト引数は、ふつうは書かずに済ませる、暗黙のものだった。C++23 では、これを最初の仮引数として明示的に書ける。本章では、この **明示的オブジェクト引数**、その型を推論して重複を減らすこと、そして自分を呼ぶラムダを扱う。

## 明示的オブジェクト引数

`Point` の二つのメンバの和を返すメンバ関数を、明示的オブジェクト引数で書く。

```cpp
struct Point
{
    int x;
    int y;
    int sum(this Point const& self) { return self.x + self.y; }
};
```

最初の仮引数に `this` を付けて、`this Point const& self` と書く。`self` がオブジェクト引数、つまり `.` の左のオブジェクトである。

```cpp
Point p = {.x = 3, .y = 4};
std::println("{}", p.sum());   // 7
```

`p.sum()` では、`self` が `p` を指す。`self.x` と `self.y` の和で `7` になる。

これは、基本編で書いた `int sum() const` と同じはたらきをする。暗黙のオブジェクト引数に付けていた `const` が、明示的オブジェクト引数では `self` の型 `Point const&` の `const` になる。基本編のオブジェクト引数の章で、C++23 ではオブジェクト引数を `this` を付けて明示的に書けると触れた。それがこれである。

## 推論で重複を減らす

明示的オブジェクト引数の型は、テンプレートにすれば推論される。テンプレートの章で見た `template <typename T>` の形で型パラメータ(ここでは `Self`)を導入し、オブジェクト引数を `this Self&& self` と書く。`Self&&` は、イテレータと範囲for の章で `auto&&` の形で見た転送参照で、`Self` は呼び出したオブジェクトから決まる。オブジェクトが const かどうか、また lvalue か rvalue かが、`Self` に反映される。本章では、const の違いに注目する。

```cpp
struct Box
{
    int value;

    template <typename Self>
    auto& get(this Self&& self) { return self.value; }
};
```

`get` は、`self` の `value` への参照を返す。`Self` が推論されるので、一つの `get` で const と非 const の両方を扱える。

```cpp
Box b = {.value = 5};
b.get() = 10;
std::println("{}", b.get());   // 10
```

非 const の `b` では、`self` は `Box&` になり、`get` は書き換えられる参照を返す。だから `b.get() = 10` で `value` を書き換えられる。

```cpp
Box const cb = {.value = 7};
std::println("{}", cb.get());   // 7
```

const の `cb` では、`self` は `Box const&` になり、`get` は読み取り専用の参照を返す。

:::message
const のオブジェクトを通して書き換える
```cpp
Box const cb = {.value = 7};
cb.get() = 9;
```
`cb` は const なので、`self` は `Box const&` に推論され、`get` は読み取り専用の参照を返す。それに代入しようとすると、ビルドに失敗する。
:::

明示的オブジェクト引数がなければ、const 版と非 const 版の `get` を別々に書くことになる。`this Self&& self` で型を推論すれば、一つにまとめられる。

:::details auto で短く書く
`this Self&& self` は、型パラメータを書かずに `auto` で短く書ける。

```cpp
auto& get(this auto&& self) { return self.value; }
```

仮引数の型に `auto` を書くのは、`template <typename Self>` で型パラメータ `Self` を導入し、その型を仮引数に書くのと同じである。`this auto&& self` は、`this Self&& self` の短い書き方になる。この、仮引数の型に `auto` を使う形は、英語では abbreviated function template と呼ばれる。ジェネリックラムダで仮引数を `auto` にしたのと書き方は似ているが、ジェネリックラムダ(C++14)が先にあり、関数の仮引数にも `auto` を使えるようにしたのが abbreviated function template(C++20)である。

`factorial` のラムダも、`[]<typename Self>` の代わりに、`[](this auto&& self, int n)` とジェネリックラムダの形で短く書ける。
:::

## 自分を呼ぶラムダ

ラムダの章で、ラムダは `operator()` を持つオブジェクトだと見た。その `operator()` のオブジェクト引数を明示的に書くと、ラムダが自分自身を `self` として受け取れる。これで、ラムダが自分を呼べる。

再帰の章の階乗を、ラムダで書く。

```cpp
auto factorial = []<typename Self>(this Self&& self, int n)
{
    if (n == 0) return 1;
    return n * self(n - 1);
};
std::println("{}", factorial(5));   // 120
```

`self` は、このラムダのクロージャ自身である。`get` と同じく、型パラメータ `Self` を導入して `this Self&& self` と書く。ラムダでは、型パラメータを `[]` に続けて `<typename Self>` と書く。`Self` は、呼び出したクロージャの型に推論される。`self(n - 1)` で、ラムダが自分を呼ぶ。`factorial(5)` は再帰の章と同じく `120` になる。
