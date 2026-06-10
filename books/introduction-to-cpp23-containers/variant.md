---
title: "いくつかの型のうち一つ"
---

`std::optional` や `std::expected` が一つの型の値を扱ったのに対し、いくつかの型のうち一つを持つのが `std::variant` である。

## いくつかの型のうち一つ

`std::variant<...>` は、挙げた型のうち、どれか一つの値を持つ。使うには `<variant>` を取り込む。言語編の共用体の章で見たとおり、共用体も複数の型のうち一つを持てたが、今どの型が入っているかは自分で覚えておく必要があった。`std::variant` は、今どの型を持っているかを自分で覚えている。

```cpp
std::variant<int, std::string> v = 5;
std::println("{}", v.index());   // 0
v = "hi";
std::println("{}", v.index());   // 1
```

`std::variant<int, std::string>` は、`int` か `std::string` のどちらか一つを持つ。`index` は、今どの型を持っているかを、挙げた順の番号で返す。`int` を持つときは `0`、`std::string` を持つときは `1` である。

:::details 他の言語の「いくつかの型のうち一つ」
「いくつかの型のうち一つを持つ」型は、他の言語にもある。共通する考え方は、タグ付き共用体、あるいは直和型(sum type)と呼ばれる。

Rust の `enum` は、各ヴァリアントがデータを抱えられる(基本編の列挙型の章で触れた)。データを持つ点で `std::variant` に近い。ただし Rust ではヴァリアントに名前が付き、`match` ですべての場合を漏れなく扱う。`std::variant` は、候補を名前でなく型で見分け、`std::visit` で扱う。

TypeScript の `A | B`(union type)も、値がいくつかの型のうち一つであることを表す。こちらは値そのものがそのいずれかの型で、`std::variant` のように包む型はない。今どの型かは、`typeof` などの絞り込みや、目印となるフィールドで見分ける。`std::variant` は、今持っている型を `index` として自分の中に持つ。
:::

## 中身を取り出す

`std::get<T>` で、`T` の値を取り出す。`std::holds_alternative<T>` は、今 `T` を持っているかを返す。

```cpp
std::variant<int, std::string> v = "hi";
if (std::holds_alternative<std::string>(v))
    std::println("{}", std::get<std::string>(v));   // hi
```

今持っている型を確かめずに別の型で取り出すと、`std::get` は `std::bad_variant_access` を投げる。確かめてから取り出すか、持ちうるすべての型をまとめて扱う `std::visit` を使う。

```cpp
std::visit([](auto const& x) { std::println("{}", x); }, v);
```

`std::visit` は、今持っている型に応じて、渡したものを呼ぶ。`auto` で受けるラムダは、`int` でも `std::string` でも呼べる(言語編のジェネリックラムダ)。どの型を持っていても、対応する処理が走る。

挙げた型の中にない型では、そもそも取り出せない。

:::message
候補にない型で取り出す
```cpp
std::variant<int, std::string> v = 5;
std::get<double>(v);
```
`double` は `std::variant<int, std::string>` の候補にない。候補にない型では取り出せず、ビルドに失敗する。
:::

`std::variant` が持てる型は、挙げた型に限られる。
