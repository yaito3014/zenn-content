---
title: "C++ でも Rust の niche optimization が欲しいので、作った話"
emoji: "🪦"
type: "tech"
topics: ["cpp"]
published: false
---

# Rust の niche optimization とは

Rust には `Option<T>` があり、そのサイズは通常 `T` のサイズの二倍になります。

```rs
fn main() {
    assert_eq!(std::mem::size_of::<i32>(), 4);
    assert_eq!(std::mem::size_of::<Option<i32>>(), 8);
}
```

これは、`enum` の variant を区別するためのタグとして 1 bit を使う際にアラインメントの関係でパディングビットが追加されてしまうことに由来します。
こうしたサイズの増大は、微々たるものではありますが不要なコストを支払ってしまうことになってしまいます。

そこで Rust では、 `&T` などヌル値が valid でない型に対してヌル値を `None` に割り当てることで `Option<&T>` のサイズを `&T` と同一のサイズにしてしまう最適化が存在します。
これは `&T` だけでなく `NonNull<T>` や `NonZero<T>` など、ヌル値を invalid とする型に対しても行われます。
このような最適化が niche optimization と呼ばれます。

# C++ の optional

C++ の `std::optional` にこういった最適化は存在せず、多くの場合不要なコストを支払い続けてしまいます。(そもそも `std::optional<T&>` が入ったのも C++26 ですし…)
そこで、特定の値が invalid であるような型に対してその値を無効値として再利用する、自称 "tombstone-aware optional" を、作りました。

https://github.com/yaito3014/polyfill/blob/84c839b00484b0a384b0e431b14e09b8f3ff37e5/include/yk/polyfill/extension/toptional.hpp

# `toptional`

今回作成した `toptional` は、 C++26 時点の `std::optional` と互換のインターフェースを持ち、かつ invalid な値を `nullopt` に割り当てて省メモリ化を図るクラスです。
