---
title: "アクセス制御"
---

クラスの章で、クラスのメンバには、外から自由に触れてよいものと、内部に隠したいものとがある、と述べた。その区別を **アクセス制御** という。本章でこれを扱う。`struct` と `class` の違いも、ここで見る。

## `public` と `private`

クラスのメンバには、クラスの外から使える **public** なものと、クラスの内側からしか使えない **private** なものがある。どちらにするかを決めるのがアクセス制御である。

数を 1 つずつ数える `Counter` を例にする。

```cpp
class Counter
{
    int count = 0;

public:
    void increment()
    {
        count = count + 1;
    }

    int value()
    {
        return count;
    }
};
```

`count` は、`public:` より前にあり、**private** なメンバである。`increment` と `value` は、`public:` の後にあり、**public** なメンバである。

private な `count` には、`Counter` の内側、すなわち `increment` や `value` の中からは触れられる。一方、`Counter` の外からは触れられない。public な `increment` と `value` は、外から呼び出せる。

データメンバ `count` には、デフォルトメンバ初期化子で `= 0` と書いてある。

```cpp
Counter c;
c.increment();
c.increment();
std::println("{}", c.value());
```

`Counter c;` は `Counter` をデフォルト初期化で作る。デフォルト初期化の章で見たように、`count` はデフォルトメンバ初期化子の `0` から始まる。`c.increment()` を 2 回呼ぶと `count` は `2` になり、`c.value()` がそれを返す。出力は `2` である。`count` に外から直接は触れず、public なメンバ関数を通してやり取りしている。

## private なメンバに外から触れる

private なメンバは、クラスの外からは使えない。使おうとすると、ビルドに失敗する。

:::message
private なメンバに外から触れる
```cpp
Counter c;
std::println("{}", c.count);
```
`count` は private なので、`Counter` の外からは触れられない。ビルドに失敗する。
:::

## `struct` と `class`

クラスを定義するキーワードには `struct` と `class` があると、クラスの章で述べた。両者の違いは、メンバの既定のアクセスだけである。`struct` では、何も指定しなければメンバは public になる。`class` では private になる。それ以外は同じである。

クラスの章で `struct` を使ったのは、メンバをすべて public にする、アクセス制御を使わない単純な場合だったからである。

## カプセル化

`Counter` では、`count` を private にして外から触れられないようにし、public な `increment` と `value` を通してのみ扱えるようにした。このように、内部のデータを隠し、外から使える操作だけを見せることを **カプセル化** と呼ぶ。

`count` に外から自由に触れられると、`Counter` は意図しない状態になりうる。たとえば、数えた覚えのない値を直接書き込まれるかもしれない。`count` を private にすれば、その変化を `increment` を通したものだけに保てる。データを隠すかどうかは、型をどう設計するかによる判断である。
