---
title: "いくつかの型のうち一つ"
---

`std::optional` や `std::expected` は、一つの型の値を、あるなし・エラーとともに表した。今度は、いくつかの型のうち一つを持つ `std::variant` を見る。

## いくつかの型のうち一つ

`std::variant<...>` は、挙げた型のうち、どれか一つの値を持つ。使うには `<variant>` を取り込む。言語編の共用体の章で見たとおり、共用体も複数の型のうち一つを持てたが、今どの型が入っているかは自分で覚えておく必要があった。`std::variant` は、今どの型を持っているかを自分で覚えている。

```cpp
std::variant<int, std::string> v = 5;
std::println("{}", v.index());   // 0
v = "hi";
std::println("{}", v.index());   // 1
```

`std::variant<int, std::string>` は、`int` か `std::string` のどちらか一つを持つ。`index` は、今どの型を持っているかを、挙げた順の番号で返す。`int` を持つときは `0`、`std::string` を持つときは `1` である。

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

`std::variant` が持てる型は、挙げた型に限られる。`int` と `std::string` を挙げたなら、その二つのうちの一つである。次章では、挙げずに、任意の型を持つ `std::any` を見る。
