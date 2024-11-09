---
title: "concat_view の内部実装について"
emoji: "🚃"
type: "tech"
topics:
  - "cpp"
published: false
---

# はじめに

まだ GCC しか `std::ranges::concat_view` を実装していない 2024 年 09 月頃、ある人の要望とひょんな思いつきから自作ライブラリに [`concat_view` の実装を生やす](https://github.com/yaito3014/yk_util/pull/36)ことになりました。
本記事は実装にあたって気になったり面白いと思ったりした箇所に関する覚書となります。

# 概要

`concat_view` は複数の `view` を連結するレンジアダプタで、受け取った view たちを `std::tuple` で内部的に保持し[^1]、イテレータはそれぞれの view のイテレータを `std::variant` で持ち[^1]つつ走査していきます。

```cpp
template <class... Views>
class concat_view {
  tuple<Views...> views_;

  class iterator {
    variant<iterator_t<Views>...> it_;
  };
};
```

[^1]: 規格上では _exposition-only_ なので実装依存ですが、簡単のため規格を参照実装として話を進めます。

ここで `std::variant` が出てくることには少し驚きましたが、メモリの節約のためでしょうか。実行時コストがどれほどになるのか気になるところではあります。

# イテレータの正規化

空の view が先頭にある時に `begin()` を呼んだとして、イテレータはいったいどの場所にあるのでしょう。空の view の `begin()` でしょうか、それとも次の空でない view の `begin()` でしょうか。
正解は後者で、 `begin()` や `operator++` の呼び出し時にイテレータの正規化を行う _satisfy_ 関数が呼ばれ、現在持っているイテレータが現在の view の `end()` と等しければ次の view に進みます。
この _satisfy_ 関数は `join_view` や `join_with_view` にも登場しますが、`concat_view` についてはテンプレートパラメータ `std::size_t N` が付いており、`N` 番目を起点として正規化を行います。

# 避けられない実行時コスト

`std::tuple` や `std::variant` の各要素へのアクセスには静的なインデックスが必要ですが、`std::variant` がどの view のイテレータを保持しているのかは実行時にしか分かりません。
規格上では

> Let i be `it_.index()`. Equivalent to: `++std::get<i>(it_);` ...

のようにごまかしていますが、実行時のインデックスを静的なインデックスに変換するためのユーティリティが必要となります。
libstdc++ の実装では `_S_invoke_with_runtime_index` や `_M_invoke_with_runtime_index` といったような名前が付いており、これは

```cpp
_S_invoke_with_runtime_index([]<std::size_t I>() {
    /* ... */
  }, runtime_index);
```

のように `std::size_t` をテンプレートパラメータに取る関数オブジェクトと実行時インデックスを渡すと、関数オブジェクトが静的なインデックスとともに呼び出されるといったものになります。
これは単純な再帰関数として実装でき、連結する view の数に比例して繰り返し回数も増えます。

# イテレータ間の距離を求める

`i`, `j` (`i` < `j`)というイテレータ間の距離を求める際には、それぞれの属する view のインデックスを `I`, `J` として

1. `I` 番目の `view` について `i`, `end()` 間の距離
2. 開区間 `(I, J)` の各 view のサイズの和
3. `J` 番目の `view` について `begin()`, `j` 間の距離

の 3 つの和を求めることになりますが、 2 番目を求める際には静的なインデックスが 2 つ必要となり、上記のユーティリティ関数を 2 段階噛ませる必要があります。
また、実際に和を求める際には `(I, J)` の連続した `std::size_t` 型の静的なインデックスが必要となるので、自前実装用には `make_index_range` なるヘルパを作ったりしました。

```cpp
// make index sequence of [I, J)
template <std::size_t I, std::size_t J>
using make_index_range = /* ... */;

static_assert(std::is_same_v<make_index_range<3, 7>, std::index_sequence<3, 4, 5, 6>>);
```

# LWG issue へ

テストケースとして `std::views::istream` を含めてテストを書いていたところ、不可解にも `concat(a, b)` が通って `concat(b, a)` が通らない事態が発生し、規格とにらめっこした末に LWG issue ([LWG issue 4166](https://cplusplus.github.io/LWG/issue4166)) を立てる運びとなりました。
当該 LWG issue は本記事の執筆時点で既に libstdc++ の実装に反映されています。

# 使い心地

連結可能性の制約についてはかなり緩く、 `char` の range と `int` の range を連結しても `std::common_type` や `std::common_reference` のおかげで `int` の range となってくれます。便利。
