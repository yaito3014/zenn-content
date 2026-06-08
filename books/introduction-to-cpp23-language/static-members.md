---
title: "static メンバ"
---

クラスのデータメンバには、オブジェクトごとに持つものと、クラスに1つだけ持つものがある。後者には `static` を付ける。本章では、クラスに1つの値を持つ `static` データメンバと、特定のオブジェクトに属さない `static` メンバ関数を扱う。

## static データメンバ

これまでのデータメンバは、オブジェクトごとに別々の値を持っていた。`static` を付けたデータメンバは、オブジェクトごとではなく、クラスに1つだけあり、すべてのオブジェクトで共有される。

`Widget` を作った回数を数える例で見る。

```cpp
struct Widget
{
    static inline int count = 0;
    int id;

    Widget() : id(count)
    {
        count = count + 1;
    }
};
```

`count` は `Widget` というクラスに1つあり、すべての `Widget` で共有される。`id` は `Widget` のオブジェクトごとにある。`Widget` を作るたびに、共有された `count` が増え、新しいオブジェクトの `id` には、そのときの `count` が入る。

```cpp
Widget a;
Widget b;
std::println("{} {} {}", a.id, b.id, Widget::count);
```

これは `0 1 2` と出力する。`id` はオブジェクトに属するので `a.id`・`b.id` と書く。`count` はオブジェクトに属さず、クラスに属するので、`Widget::count` とクラス名で指す。

`count` には `static` とともに `inline` を付け、`= 0` と初期値を書いた。この `inline` は、ODR と inline の章で見たものである。`static` データメンバは、宣言とは別に定義を1つ必要とするが、`inline` を付けると、この場の初期化がそのまま定義になり、別に定義を書かずに済む。

:::details inline を付けない static データメンバ
`inline` を付けずに `static int count;` とクラスの中に書くと、それは宣言であって定義ではない。別のソースに `int Widget::count = 0;` と定義を1つ書く必要があり、書かないとリンクに失敗する。`static inline int count = 0;` は、この別の定義を不要にする書き方である。
:::

## static メンバ関数

`static` は、メンバ関数にも付けられる。`static` メンバ関数は、特定のオブジェクトにではなくクラスに属し、オブジェクト引数(`.` の左のオブジェクト)を取らない。先ほどの `Widget` に、作った数を返す `static` メンバ関数 `how_many` を加える。

```cpp
struct Widget
{
    static inline int count = 0;
    int id;

    Widget() : id(count) { count = count + 1; }

    static int how_many()
    {
        return count;
    }
};
```

`how_many` はオブジェクトを通さず、`Widget::how_many()` と呼ぶ。

```cpp
Widget a;
Widget b;
std::println("{}", Widget::how_many());
```

これは `2` と出力する。

非 `static` のデータメンバは特定のオブジェクトに属し、使うにはどのオブジェクトのものかが要る。`static` メンバ関数はオブジェクト引数を取らないので、その「どのオブジェクト」が定まらず、非 `static` のデータメンバは使えない。クラスに属する `count` のような `static` データメンバは使える。

:::message
static メンバ関数から非 static のデータメンバを使う
```cpp
static int how_many()
{
    return id;
}
```
`id` はオブジェクトごとのデータメンバである。`static` メンバ関数はオブジェクト引数を取らないため、どのオブジェクトの `id` かが定まらず、ビルドに失敗する。
:::
