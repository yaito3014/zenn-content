---
title: "オブジェクト引数と const メンバ関数"
---

型と関数の章では、関数を引数の型と戻り値の型から見た。だがクラスの章のメンバ関数 `area` は、引数を一つも書いていないのに、`Rectangle` のデータメンバ `width` と `height` を使えた。アクセス制御の章の `value` も、引数なしで `count` を返せた。この「書いていない引数」が **オブジェクト引数** である。本章ではこれを捉えたうえで、それに `const` を付ける **const メンバ関数** と、書き換えない操作を const にする考え方を扱う。

## オブジェクト引数

クラスの章で定義した `Rectangle` の `area` を振り返る。

```cpp
struct Rectangle
{
    int width;
    int height;

    int area()
    {
        return width * height;
    }
};
```

`area` の仮引数の並びは空である。それでも本体では `width` と `height` を使っている。

```cpp
Rectangle r = {.width = 3, .height = 4};
r.area();
```

`r.area()` と呼ぶと、`area` の本体の `width` と `height` は、`.` の左に書いたオブジェクト `r` のデータメンバ、すなわち `r` の `width` と `height` を指す。メンバ関数は、書かれた仮引数とは別に、`.` の左のオブジェクトを受け取る。この受け取る対象を **オブジェクト引数** と呼ぶ。`r.area()` では、`.` の左の `r` がオブジェクト引数になる。

型と関数の章で、シグネチャはメンバ関数では属するクラスや `const` などの修飾が加わる、と触れた。その `const` は、このオブジェクト引数に効く。

:::details 規格でのオブジェクト引数
メンバ関数の呼び出しでどの関数を選ぶか(オーバーロード解決)を決めるとき、規格はメンバ関数が **暗黙のオブジェクト引数**(implicit object parameter)を一つ余分に持つものとして扱う。本章のオブジェクト引数はこれを指す。C++23 では、この対象を `this` を付けた仮引数として明示的に書くこともできる。本書では明示的には書かず、暗黙の側だけを扱う。
:::

## const メンバ関数

`area` は `width` と `height` を読むだけで、オブジェクトを書き換えない。書き換えないメンバ関数には、そのことを `const` で示せる。

```cpp
int area() const
{
    return width * height;
}
```

仮引数の並びの後ろに `const` を付ける。これは、オブジェクト引数を通してオブジェクトを書き換えない、という約束である。参照とポインタの章で見た const 参照を思い出すとよい。const 参照を通してオブジェクトを書き換えられなかったのと同じく、const メンバ関数の中では、オブジェクト引数のデータメンバを書き換えられない。

:::message
const メンバ関数の中でデータメンバを書き換える
```cpp
int area() const
{
    width = 0;
    return width * height;
}
```
`area` は const メンバ関数なので、オブジェクト引数を通して `width` を書き換えることはできない。ビルドに失敗する。
:::

const を付けるかどうかは、呼び出せる相手にも関わる。const なオブジェクトや、const 参照の先では、const メンバ関数しか呼べない。参照とポインタの章では、`Rectangle` を読み取るだけの関数を const 参照で受け取った。同じ形で `area` を呼ぶ場合を考える。

```cpp
void print_area(Rectangle const& r)
{
    std::println("{}", r.area());
}
```

`r` は const 参照なので、`r` を通して呼べるのは const メンバ関数だけである。`area` に `const` を付けていなければ、ここで `r.area()` は呼び出せず、ビルドに失敗する。読み取るだけの `area` を const にしておくと、こうした const 参照の先でも使える。

## 書き換えない操作に const を付ける

書き換えない操作には const を付け、書き換える操作には付けない。`area` を const にしたのも、読むだけだからである。同じ見方で、アクセス制御の章の `Counter` も見直せる。`value` は `count` を返すだけで書き換えないので const にし、`increment` は `count` を書き換えるので付けない。

```cpp
class Counter
{
    int count = 0;

public:
    void increment()
    {
        count = count + 1;
    }

    int value() const
    {
        return count;
    }
};
```

`value` に `const` を付けた。`increment` は `count` を書き換えるので、付けない。読み取るだけの操作を const にし、書き換える操作と区別するこの使い分けは、英語では const correctness と呼ばれる。本書も、メンバ関数を書くときの基本としてこれに従う。

クラスの章の `area` も、アクセス制御の章の `value` も、これまで const を付けずにきた。どちらも読み取るだけの操作なので、本来は const にすべきものだった。const メンバ関数をここで導入したので、両方に揃えて const を付ける。
