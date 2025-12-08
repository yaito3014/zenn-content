---
title: "C++26リフレクションでつくる tuple と variant"
emoji: "✨️"
type: "tech"
topics: ["cpp"]
published: true
---

# はじめに

この記事は [C++ Advent Calendar 2025](https://qiita.com/advent-calendar/2025/cxx) 9 日目の記事となります。
C++26 には静的リフレクション([P2996R13 Reflection for C++26](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2996r13.html))が採択されました。
この変更は今まで行われてきたメタプログラミングを凌駕する表現の可能性を秘めています。
[magic_enum](https://github.com/Neargye/magic_enum) 相当の処理が標準の範囲内でできるようになるということは既に有名かもしれません。

# 既存の tuple や variant

本題に入る前に、既存の tuple や variant がどのように実装されてきたのかを見ていきたいと思います。

## tuple

まずは tuple から見ていきたいのですが、その前に少し pair の話を。
`std::tuple` が登場する以前から `std::pair` という型がありました。 `std::pair` は `std::map` や `std::unordered_map` といった連想コンテナのエントリとして使われましたが、現代の C++ からすれば std::tuple か named struct で置き換え可能なものとなっています。
`std::pair` の定義は至ってシンプルで、データメンバだけ並べるなら以下のようになります。

```cpp
template<class F, class S>
struct pair {
  F first;
  S second;
};
```

ここからより一般的な tuple に拡張するにはどうすればよいでしょう。
一つの答えは同じ構造のまま定義を増やすことです。

```cpp
struct tuple0 {};

template<class T1>
struct tuple1 {
  T1 value1;
};

template<class T1, class T2>
struct tuple2 {
  T1 value1;
  T2 value2;
};

template<class T1, class T2, class T3>
struct tuple3 {
  T1 value1;
  T2 value2;
  T3 value3;
};

// ...
```

とても煩雑ですが、 可変長引数テンプレートのない C++11 未満では致し方ありません[^pre-c++11]。

[^pre-c++11]: より詳しくは [Boost.MPL](https://www.boost.org/doc/libs/latest/libs/mpl/doc/index.html) という遺物を参照のこと

では可変長引数テンプレートさえあれば容易に実装可能かと言われれば、そうでもありません。
理想的には

```cpp
template<class... Ts>
struct tuple {
  Ts... values;
};
```

と書きたいところですが、パックを直接保持することを C++ 標準は許さないようで、これはできません。[^member-pack]

[^member-pack]: これを許可しようという提案はされています
[P3115R0 Data Members, Variables and Alias Declarations Can Introduce A Pack](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3115r0.pdf)

本記事ではメジャーであろう 2 通りの実装を示します。

### tuple 実装その 1

まずひとつは、値の列を先頭の一つと残りに分けて再帰的に保持していく実装方法です。

```cpp
template<std::size_t I, class... Ts>
struct tuple_impl;

template<std::size_t I>
struct tuple_impl<I> {};

template<std::size_t I, class Head, class... Rest>
struct tuple_impl<I, Head, Rest...> {
  Head head;
  tuple_impl<I + 1, Rest...> rest;
};

template<class... Ts>
struct tuple : tuple_impl<0, Ts...> {};
```

すぐに思いつきやすいのはこちらでしょうか。素直な実装となっています。
libstdc++ や microsoft/STL の実装がこれにあたります。

### tuple 実装その 2

もうひとつは、インデックスを型に埋め込みつつ値を保持するヘルパ型を一気に継承してしまう実装方法です。

```cpp
template<std::size_t I, class T>
struct tuple_leaf {
  T value;
};

template<class IndexSequence, class... Ts>
struct tuple_impl;

template<std::size_t... Is, class... Ts>
struct tuple_impl<std::index_sequence<Is...>, Ts...> : tuple_leaf<Is, Ts>... {};

template<class... Ts>
struct tuple : tuple_impl<std::index_sequence_for<Ts...>, Ts...> {};
```

C++ で複数のクラスを継承できることを利用したクレバーな実装です。
libc++ の実装がこれにあたります。

## variant

次に variant です。そもそも variant は古来からの tagged union をテンプレートを駆使して構成したものになっています。
こちらも理想的には

```cpp
template<class... Ts>
struct variant {
  int tag;
  union {
    Ts... alternatives
  };
};
```

と書きたいところですが、こちらも同様の理由でできません。

`std::variant` の実装は libstdc++, libc++, microsoft/STL のいずれでもほぼ一貫しており、

```cpp
template<class... Ts>
struct variadic_union;

template<>
struct variadic_union<> {};

template<class Head, class... Rest>
struct variadic_union<Head, Rest...> {
  union {
    Head head;
    variadic_union<Rest...> rest;
  };
};

template<class... Ts>
struct variant {
  int tag;
  variadic_union<Ts...> storage;
};
```

のような再帰的な定義になっています。

# リフレクションでできること

ここでリフレクションの話に入ります。 本記事で扱う C++26 リフレクションの要点を挙げると

1. あらゆる entity の情報を保持する型 `std::meta::info`
2. `std::meta::info` を**得る**リフレクション演算子 `^^`
3. `std::meta::info` を取り巻く consteval 関数群

となります。今回は

> 3. `std::meta::info` を取り巻く consteval 関数群

の中でも `std::meta::define_aggregate` による aggregate の定義に焦点を当てます。
`std::meta::define_aggregate` はその時点で incomplete なクラス型の _reflection_ とデータメンバの持つべき性質を表す _data member description_ [^dmd] という種類の _reflection_ の列を受け取りそのクラス型を aggregate として定義します。

[^dmd]: 型、識別子、アラインメント、ビットフィールドの幅、no_unique_address かどうかを表す boolean の５つ組として定義されています。詳しくは [規格](https://eel.is/c++draft/class.mem.general#def:description,data_member) を参照のこと。

概要だけでは分かりづらいと思うので、具体例を示します。

```cpp
struct my_class;

// my_class is incomplete type here

consteval {
  const auto member1 = std::meta::data_member_spec(^^int, { .name = "foo" });
  const auto member2 = std::meta::data_member_spec(^^double, { .name = "bar" });
  std::meta::define_aggregate(^^my_class, { member1, member2 });
}

// my_class is complete type here
```

上のようにすると `my_class` は以下のような定義を持ったのと同様に定義されます。

```cpp
struct my_class {
  int foo;
  double bar;
};
```

ここで登場した `consteval { /* ... */ }` は consteval block と呼ばれるもので、トップレベルで評価される複合文を置くことが出来ます。
`std::meta::data_member_spec` 関数は型の _reflection_ (ここでは `^^int` や `^^double`) を第一引数に取り、`std::meta::data_member_options` を第二引数に取って _data member description_ を返します。
`std::meta::data_member_options` は概ね以下の構造をした `std::meta::data_member_spec` 用のヘルパ型です。

```cpp
struct data_member_options {
  struct name-type;  // exposition-only
  std::optional<name-type> name;
  std::optional<int> alignment;
  std::optional<int> bit_width;
  bool no_unique_address = false;
};
```

宣言するデータメンバの持つべき性質を表すシンプルな型となっています。
各メンバが `std::optional` で表現されていますが、実際には `name` と `bit_width` の少なくともどちらか一方は値を持たなければいけません。

ここで

```cpp
struct my_class {
  consteval {
    std::meta::define_aggregate(^^my_class, { /* ... */ });
  }
};
```

とは出来ないことに注意しましょう。`std::meta::define_aggregate` を、まさに定義中のクラス内で行うことは出来ません。

## tuple

`std::meta::define_aggregate` には任意の _data member description_ 列を渡すことができるので、コンパイル時に定まる限り任意個のデータメンバを指定できます。
なんとこれは tuple にぴったりです。
`constexpr` な `to_string` の存在を仮定すれば [^constexpr-to-string]

[^constexpr-to-string]: C++23 から `constexpr` となった `std::to_chars` を使えば容易に実装可能です。

```cpp
template<class... Ts>
struct tuple;

consteval {
  const auto data_member_options =
    std::vector{ ^^Ts... }
    | std::views::enumerate
    | std::views::transform([](auto t) {
        return std::meta::data_member_spec(std::get<1>(t), { .name = "_" + to_string(std::get<0>(t)) });
      });
  std::meta::define_aggregate(^^tuple<Ts...>, data_member_options);
}
```

と書きたいのですが、ここでどこから `Ts` を取ってくるべきか悩みます。
前述したように

```cpp
template<class... Ts>
struct tuple {
  consteval {
    const auto data_member_options = /* ... */
    std::meta::define_aggregate(^^tuple, {});
  }
};
```

と定義中のクラスを定義する事もできません。
そのため一度メンバ型を挟んで、改めてそれを使う必要があります。

```diff cpp
template<class... Ts>
struct tuple {
  struct storage;

  consteval {
    const auto data_member_options =
      std::vector{ ^^Ts... }
      | std::views::enumerate
      | std::views::transform([](auto t) {
          return std::meta::data_member_spec(std::get<1>(t), { .name = "_" + to_string(std::get<0>(t)) });
        });
-   std::meta::define_aggregate(^^tuple, data_member_options);
+   std::meta::define_aggregate(^^storage, data_member_options);
  }

  storage data;
};
```

以上のように定義した `tuple` の特殊化 `tuple<int, double>` は以下と同様の構造になります

```cpp
struct tuple_int_double {
  struct storage {
    int _0;
    double _1;
  } data;
};
```

適宜コンストラクタや `get` フリー関数 [^get] などを生やしてあげればあっという間に立派な tuple になることでしょう。

[^get]: `std::meta::nonstatic_data_members_of` 関数でクラスに対して非静的データメンバをクエリできます

## variant

tuple と実はほぼ同じコードで実現できます。 `struct storage` の代わりに `union storage` としてタグ型を追加するだけです。

```diff cpp
  template<class... Ts>
- struct tuple {
+ struct variant {
-   struct storage;
+   union storage;

    consteval {
      const auto data_member_options =
        std::vector{ ^^Ts... }
        | std::views::enumerate
        | std::views::transform([](auto t) {
            return std::meta::data_member_spec(std::get<1>(t), { .name = "_" + to_string(std::get<0>(t)) });
          });
      std::meta::define_aggregate(^^storage, data_member_options);
    }

+   int tag;
    storage data;
  };
```

これも同様に、 `variant<int, double>` が以下と同じ構造になります

```cpp
struct variant_int_double {
  int tag;
  union storage {
    int _0;
    double _1;
  } data;
};
```

# おわりに

C++26 のリフレクションが主要な処理系で実装されるのを楽しみにしています。

[GCC の reflection 実験的実装での動作例](https://godbolt.org/z/zPxGbGE6n)

# 補足

- Q. リフレクション、速いの？
  - A. まだ計測していません。多分速いんじゃないかなとは思いつつ処理系の実装が安定するのを待ちましょう。
- Q. 標準の実装もシンプルになる？
  - A. 真の難しさはメンバ関数に加わる制約([_Constraints_:](https://eel.is/c++draft/library#structure.specifications-3.1), [_Mandates_:](https://eel.is/c++draft/library#structure.specifications-3.2), etc)などを含めて「正しく」実装することにあります。
    C++26 ベースで書ける前提なら少しは簡単になるかもしれませんが、劇的にコードが小さくなるということはないと思います。
- Q. リフレクションのエラーハンドリングは？
  - A. `std::meta::exception` が投げられたり、定数式でなくなったりします。後者は規格内で [_Constant When_:](https://eel.is/c++draft/library#structure.specifications-3.3) のような節が新設されています。
