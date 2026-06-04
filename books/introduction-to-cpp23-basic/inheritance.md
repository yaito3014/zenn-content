---
title: "継承"
---

クラスとアクセス制御の章で、自分で型を定義し、メンバのアクセスを制御した。本章では、既存のクラスをもとに新しいクラスを作る **継承** を扱う。アクセス制御の章で名前だけ挙げた **protected** と、`struct` と `class` の継承における既定のアクセスの違いも、ここで見る。

## 派生クラス

これまで見てきた `Counter` を振り返る。

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

この `Counter` に、数えた値を `0` に戻す `reset` を加えた新しい型を作りたいとする。`Counter` を書き換えずに、それをもとにした型を作れる。これを **継承** と呼ぶ。次のように書く。

```cpp
class ResettableCounter : public Counter
{
public:
    void reset()
    {
        count = 0;
    }
};
```

`: public Counter` は、`Counter` を継承するという意味である。`ResettableCounter` は `Counter` のメンバ(`increment` と `value`)をそのまま受け継ぎ、さらに `reset` を加える。もとになる `Counter` を **基底クラス**、それを継承した `ResettableCounter` を **派生クラス** と呼ぶ。

ただし、このままではビルドに失敗する。`count` は `Counter` の private なメンバで、private なメンバは `Counter` 自身のメンバからしか使えない。派生クラスの `ResettableCounter` のメンバは、これに含まれない。だから `reset` から `count` には触れられない。

:::message
派生クラスから基底クラスの private メンバに触れる
```cpp
void reset()
{
    count = 0;
}
```
`reset` は `ResettableCounter` のメンバだが、`count` は基底クラス `Counter` の private なメンバなので、ここからは触れられない。ビルドに失敗する。
:::

## protected

派生クラスから触れられるようにするには、そのメンバを **protected** にする。`Counter` の `count` を protected にする。

```cpp
class Counter
{
protected:
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

protected なメンバは、そのクラス自身のメンバに加えて、派生クラスのメンバからも触れられる。これで `ResettableCounter` の `reset` から `count` を使える。

```cpp
ResettableCounter c;
c.increment();
c.increment();
std::println("{}", c.value());
c.reset();
std::println("{}", c.value());
```

`c.increment()` を2回呼ぶと `count` は `2` になり、最初の `c.value()` は `2` を返す。`c.reset()` で `count` は `0` に戻り、次の `c.value()` は `0` を返す。出力は `2`、続いて `0` になる。`increment` と `value` は `Counter` から受け継いだもの、`reset` は `ResettableCounter` で加えたものである。

ただし protected は public とは違う。protected なメンバに触れられるのは、`Counter` 自身のメンバと派生クラスのメンバに限られる。public のように、どこからでも触れられるわけではない。

:::message
protected なメンバに使う側から触れる
```cpp
ResettableCounter c;
c.count = 0;
```
`count` は protected なので、`ResettableCounter` を使う側のコードからは触れられない。ビルドに失敗する。
:::

protected は、使う側から隠す点では private と同じで、派生クラスから使える点が private と違う。public と private の中間にあたるアクセスである。

## 継承の既定アクセス

`: public Counter` の `public` は、継承のしかたを指定するものである。これを **public 継承** と呼ぶ。public 継承では、基底クラスの public なメンバは派生クラスでも public のままで、使う側から呼べる。だから先ほどの例で `c.increment()` や `c.value()` を呼べた。

この `public` は省くこともでき、省いたときの既定は `struct` と `class` で異なる。`struct` では public 継承、`class` では private 継承が既定になる。private 継承では、基底クラスの public なメンバが派生クラスでは private になり、使う側から使えなくなる。

メンバの既定アクセスが `struct` と `class` で違ったのと同じく、継承の既定アクセスも違う。アクセス制御の章で「継承においても既定のアクセスが異なる」と述べたのは、この点である。

`class` で `public` を省くと private 継承になり、`increment` や `value` を使う側から呼べなくなる。継承のアクセスは省略時の既定に頼らず、`: public` のように明示しておけば、こうした取り違えを防げる。本書では明示する。

継承のもう一つの使い道に、基底クラスの参照やポインタを通して、派生クラスごとに異なる振る舞いを呼び分ける、**仮想関数** を使ったしくみがある。これは別の章で扱う。
