---
title: "仮想関数"
---

継承の章では、`Counter` をもとに派生クラスを作った。本章では、仮想関数を使って、基底クラスの参照やポインタを通して、派生クラスごとに異なる振る舞いを呼び分けるしくみを扱う。

## 仮想関数とオーバーライド

継承の章で `count` を protected にした `Counter` をもとにする。今度は、数え方を派生クラスごとに変えられるようにしたい。そのために、`increment` を **仮想関数** にする。関数の前に `virtual` を付ける。

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
};
```

`virtual` を付けた `increment` は、派生クラスで定義し直せる。`step` ずつ数える `StepCounter` を派生クラスとして作る。

```cpp
class StepCounter : public Counter
{
    int step = 1;

public:
    explicit StepCounter(int s) : step(s)
    {
    }

    void increment() override
    {
        count = count + step;
    }
};
```

`StepCounter` の `increment` は、`count` を `step` ずつ増やす。基底クラスの仮想関数を派生クラスで定義し直すことを **オーバーライド** と呼ぶ。関数の後ろに付けた `override` は、これが基底クラスの仮想関数をオーバーライドすることを示す。

## 基底クラスの参照を通して呼び分ける

基底クラス `Counter` の参照を受け取る関数を考える。

```cpp
void count_up(Counter& c)
{
    c.increment();
    c.increment();
}
```

`count_up` は `Counter&` を受け取り、`increment` を2回呼ぶ。`StepCounter` は `Counter` を public 継承している。基底クラスの参照は、その派生クラスのオブジェクトも指せるので、`Counter&` を取る `count_up` には、`Counter` も `StepCounter` も渡せる。

```cpp
Counter a;
count_up(a);
std::println("{}", a.value());

StepCounter b(10);
count_up(b);
std::println("{}", b.value());
```

`count_up(a)` では `a` の `increment` が2回呼ばれ、`count` は `2` になる。`count_up(b)` では `b` の `increment`、すなわち `StepCounter` でオーバーライドしたほうが呼ばれ、`count` は `10` ずつ増えて `20` になる。出力は `2`、続いて `20` である。

同じ `count_up` の同じ `c.increment()` でも、`c` が指しているオブジェクトが `Counter` か `StepCounter` かによって、呼ばれる `increment` が変わる。この、指している先のオブジェクトの型を **動的な型** と呼ぶ。仮想関数は、動的な型に応じて呼び分けられる。動的な型が派生クラスのときは、派生クラスでオーバーライドした版が、基底クラスの版に優先して呼ばれる。

`increment` を `virtual` にしていなければ、`count_up` の中の `c.increment()` は、つねに `Counter` の `increment` になる。`StepCounter` を渡してもオーバーライドしたほうは呼ばれず、`a` も `b` も同じ振る舞いになる。

## override でオーバーライドの取り違えを防ぐ

`override` を付けておくと、オーバーライドのつもりがオーバーライドになっていないとき、ビルドに失敗する。たとえば `increment` の綴りを間違えると、基底クラスに対応する仮想関数がない。

:::message
override を付けた関数が何もオーバーライドしていない
```cpp
void incrment() override
{
    count = count + step;
}
```
`incrment` は綴り間違いで、基底クラス `Counter` にこの名前の仮想関数はない。`override` を付けているため、ビルドに失敗する。
:::

もし `override` を付けていなければ、これは `increment` とは別の新しい関数とみなされ、ビルドは成功してしまう。そのうえ、`Counter&` を通した `increment` の呼び出しは基底クラスのままになり、オーバーライドしたつもりの関数は呼ばれない。`override` は、こうしたビルドの通る誤りを、ビルドの段階の誤りに変える。

## 値で受け取るとスライシングが起きる

仮想関数による呼び分けには、`count_up` のように基底クラスの参照(またはポインタ)で扱うことが要る。基底クラスの値にコピーすると、派生クラスの部分が落ちてしまう。

```cpp
StepCounter b(10);
Counter c = b;
c.increment();
std::println("{}", c.value());
```

`Counter c = b;` で `b` を `Counter` にコピーすると、`Counter` の部分だけが移り、`StepCounter` の部分は落ちる。`c` は `Counter` なので、`c.increment()` は `Counter` の `increment` になり、`count` は `1` になる。`StepCounter` の数え方は失われている。派生クラスの部分が落ちるこの現象を **スライシング** と呼ぶ。仮想関数を効かせるには、値ではなく参照やポインタで扱う。

## 仮想デストラクタ

本書では、仮想関数を持つ基底クラスのデストラクタを `virtual` にする。基底クラスのポインタを通してオブジェクトを破棄する場面で関わるためで、その場面は `new` と `delete` を扱う後の章で見る。
