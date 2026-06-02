---
title: "動的なオブジェクト"
---

これまで、オブジェクトは宣言した場所で作られ、そのブロックを抜けると自動で破棄された。寿命がブロックに結びついていた。だが、寿命を自分で決めて、好きなときに作り、好きなときに壊したい場合がある。それを `new` と `delete` で行う。本章では、`new`/`delete` による動的なオブジェクトの確保と破棄、そこで効く仮想デストラクタ、そして確保と破棄を引き受ける標準ライブラリの道具への橋渡しを扱う。

## new と delete

`new` で、オブジェクトを動的に作る。

```cpp
Counter* p = new Counter();
p->increment();
p->increment();
std::println("{}", p->value());
delete p;
```

`new Counter()` は、`Counter` のオブジェクトを動的に1つ作り、そのアドレスを返す。`Counter*` の `p` でそのアドレスを受け取る。`p->increment()` は、`p` の指す先のメンバ関数を呼ぶ書き方で、`(*p).increment()` と同じである。使い終わったら `delete p;` で、`p` の指すオブジェクトを破棄する。出力は `2` である。

`new` で作ったオブジェクトは、ブロックを抜けても自動では破棄されない。`delete` を書いたときに破棄される。寿命を自分で管理することになる。

## delete を忘れるとリークする

`new` で作ったオブジェクトを `delete` で破棄しないと、使い終わってもそのまま残り続ける。確保した領域が解放されず、使われないまま占有される。これを **メモリリーク** と呼ぶ。`new` を書いたら、対応する `delete` を書かなければならない。

リークはビルドを止めず、すぐに目に見える問題も起こさないことが多い。だが積み重なれば、使える領域を少しずつ食いつぶす。

## 基底クラスのポインタ経由の破棄

多態の章で、多態のために継承する基底クラスのデストラクタを `virtual` にすると述べた。その理由が、この `delete` にある。

動的に作った派生クラスのオブジェクトを、基底クラスのポインタを通して破棄することがある。そのために、基底クラス `Counter` のデストラクタを `virtual` にしておく。

```cpp
class Counter
{
protected:
    int count = 0;

public:
    virtual void increment()
    {
        count = count + 1;
    }

    int value() const
    {
        return count;
    }

    virtual ~Counter()
    {
    }
};
```

`virtual ~Counter()` は、本体では何もしないデストラクタだが、`virtual` を付けてあることに意味がある。

```cpp
Counter* p = new StepCounter(10);
delete p;
```

`p` は `Counter*` だが、指しているのは `StepCounter` のオブジェクトである。`Counter` のデストラクタが `virtual` なので、`delete p;` は動的な型 `StepCounter` のデストラクタを呼ぶ。

:::message alert
仮想でないデストラクタを基底クラスのポインタ経由で delete する
```cpp
Counter* p = new StepCounter(10);
delete p;
```
`Counter` のデストラクタが `virtual` でないと、`Counter*` を通した `delete` では `StepCounter` のデストラクタが呼ばれない。結果は定まらず、実行時に深刻な問題が起こりうる。
:::

多態のために継承する基底クラスのデストラクタを `virtual` にするのは、この破棄を正しく行うためである。

## 確保と破棄を引き受ける道具へ

`new` と `delete` を正しく対にするのは、書く側の責任になる。`delete` を忘れればリークし、すでに `delete` したオブジェクトをもう一度 `delete` すれば、やはり結果は定まらない。手動の管理は誤りを生みやすい。

そこで、確保と破棄を引き受ける道具が標準ライブラリにある。`std::unique_ptr` や `std::shared_ptr` は、`new` で作ったオブジェクトを持ち、自身の寿命が尽きるときに自動で `delete` する。`new` と `delete` を自分で対にして書く代わりに、こうした道具を使う。これらは第3部で扱う。
