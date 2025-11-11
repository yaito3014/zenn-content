---
title: "std::type_orderによる型リストの正規化"
emoji: "📏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cpp"]
published: true
---

# `std::type_order` について

C++26 において `std::type_order` クラステンプレートが導入されます[^1]。これは２つの型を取って `std::strong_ordering` を返すようなメタ関数であり、ユーザー定義型を含めあらゆる型に対し処理系定義の一貫した順序付けを行うものです。

[^1]: 2025/11/11 現在 GCC の libstdc++ のみが実装しています

```cpp
struct A {};
struct B {};

constexpr std::strong_ordering ord = std::type_order<A, B>::value; // 処理系定義の値
```

これまでにも `std::type_info` による実行時型情報を利用した比較[^2]や `boost::mp11::mp_less` による `value` メンバの比較など型に対する比較自体は存在したのですが、「コンパイル時に」「型だけを直接」比較する手段は標準の機能の中では存在しませんでした。標準を外れるならば `__PRETTY_FUNCTION__` にテンプレートパラメータが埋め込まれることを利用した比較もできましたが、依然として一貫性に問題がありました[^3]。

[^2]: `std::type_order` の順序は `std::type_info::before` と一貫性があることを保証しません
[^3]: 例えば enum class の定義が visible かどうかで埋め込まれる文字列が異なってしまいます

# 型リストの正規化

型の順序付けができると型リストに対するソートができるようになり、そして正規化に繋がります。ここでいう型の正規化というのは、 TypeScript におけるユニオン型の挙動に近いものです。TypeScript では `A | B` と `B | A` は同じ型ですし、`(A | B) | C` と `A | (B | C)` も同じ型です。しかしこれを愚直に C++ に訳せば `std::variant<A, B>` と `std::variant<B, A>` 及び `std::variant<std::variant<A, B>, C>` と `std::variant<A, std::variant<B, C>>` といった、それぞれ異なる型となってしまいます。後者に関しては `std::variant` を「折りたたむ」ことで `std::variant<A, B, C>` にすることができますが、型の順序の違いや重複の有無に関しては型が区別されてしまうままです。
これらの違いを吸収するためには型リストを `std::variant` に渡す前に正規化する必要があります。ここで `apply_canonicalized` というエイリアステンプレートを以下のように定義します。

```cpp
template<template<class...> class TT, class.... Ts>
using apply_canonicalized = TT</* sorted + uniqued Ts */>;
```

これは `Ts` をソート及び重複削除したものを `TT` に適用するようなもので、以下のような性質を持ちます。

```cpp
struct A {};
struct B {};

template<class... Ts>
struct type_list {};

static_assert(std::is_same_v<
  apply_canonicalized<type_list, A, A, A>,
  type_list<A>
>);

static_assert(std::is_same_v<
  apply_canonicalized<type_list, A, B>,
  apply_canonicalized<type_list, B, A>
>);
```

これを用いると、事前に型リストを正規化した `std::variant` を生成するような `variant_for` を定義できます。

```cpp
template<class... Ts>
using variant_for = apply_canonicalized<std::variant, Ts...>;
```

これに `std::variant` を「折りたたむ」機構を組み込むことで、 TypeScript のユニオン型と同じような挙動にすることができます。

```cpp
template<class... Ts>
struct type_list {};

template<class T>
struct unwrap_single_variant {
  using type = T;
};

template<class T>
struct unwrap_single_variant<std::variant<T>> {
  using type = T;
};

template<class T>
struct expand_if_variant {
  using type = type_list<T>;
};

template<class... Ts>
struct expand_if_variant<std::variant<Ts...>> {
  using type = type_list<Ts...>;
};

template<class ResultTypeList, class... TypeLists>
struct flatten_impl;

template<class... Ts>
struct flatten_impl<type_list<Ts...>> {
  using type = type_list<Ts...>;
};

template<class... Ts, class U, class... TypeLists>
struct flatten_impl<type_list<Ts...>, U, TypeLists...> : flatten_impl<type_list<Ts..., U>, TypeLists...> {};

template<class... Ts, class... Us, class... TypeLists>
struct flatten_impl<type_list<Ts...>, type_list<Us...>, TypeLists...> : flatten_impl<type_list<Ts..., Us...>, TypeLists...> {};

template<class... TypeLists>
struct flatten : flatten_impl<type_list<>, TypeLists...> {};

template<template<class...> class TT, class TypeList>
struct apply;

template<template<class...> class TT, class... Ts>
struct apply<TT, type_list<Ts...>> {
  using type = TT<Ts...>;
};

template<class... Ts>
struct unite : unwrap_single_variant<typename apply<variant_for, typename flatten<typename expand_if_variant<Ts>::type...>::type>::type> {};

template<class... Ts>
using unite_t = typename unite<Ts...>::type;

static_assert(std::is_same_v<
  unite_t<A, B>,
  variant_for<A, B>
>);

static_assert(std::is_same_v<
  unite_t<B, A>,
  variant_for<A, B>
>);

static_assert(std::is_same_v<
  unite_t<A, std::variant<B, C>>,
  variant_for<A, B, C>
>);

static_assert(std::is_same_v<
  unite_t<std::variant<A, B>, C>,
  variant_for<A, B, C>
>);
```

こうして定義した `unite` を利用すれば、 `std::variant` が関わる関数のシグネチャなどを同じ型の集合に対して同一にすることができます。
