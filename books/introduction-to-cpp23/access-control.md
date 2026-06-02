---
title: "アクセス制御"
---

クラスの章で、クラスのメンバには、どこからでも自由に触れてよいものと、隠しておきたいものとがある、と述べた。その区別を **アクセス制御** という。本章でこれを扱う。`struct` と `class` の違いも、ここで見る。

## `public` と `private`

クラスのメンバには、どこからでも使える **public** なものと、そのクラスのメンバ関数からしか使えない **private** なものがある。メンバをどう見せるかを決めるのがアクセス制御である。これらのほかに **protected** もある。これは継承に関わるもので、後の章で触れる。

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

private な `count` には、`Counter` のメンバ関数、すなわち `increment` や `value` の中からは触れられる。一方、`Counter` を使う側からは触れられない。public な `increment` と `value` は、使う側から呼び出せる。

データメンバ `count` には、デフォルトメンバ初期化子で `= 0` と書いてある。

```cpp
Counter c;
c.increment();
c.increment();
std::println("{}", c.value());
```

`Counter c;` は `Counter` をデフォルト初期化で作る。デフォルト初期化の章で見たように、`count` はデフォルトメンバ初期化子の `0` から始まる。`c.increment()` を 2 回呼ぶと `count` は `2` になり、`c.value()` がそれを返す。出力は `2` である。`count` に使う側から直接は触れず、public なメンバ関数を通してやり取りしている。

## private なメンバに使う側から触れる

private なメンバは、使う側からは使えない。使おうとすると、ビルドに失敗する。

:::message
private なメンバに使う側から触れる
```cpp
Counter c;
std::println("{}", c.count);
```
`count` は private なので、`Counter` を使う側からは触れられない。ビルドに失敗する。
:::

## `struct` と `class`

クラスを定義するキーワードには `struct` と `class` があると、クラスの章で述べた。両者の違いは、既定のアクセスにある。メンバについては、`struct` では何も指定しなければ public、`class` では private になる。継承においても既定のアクセスが異なるが、それは継承の章で扱う。この違いを除けば、`struct` と `class` は同じものである。

クラスの章で `struct` を使ったのは、メンバをすべて public にする、アクセス制御を使わない単純な場合だったからである。

## カプセル化

`Counter` では、`count` を private にして使う側から触れられないようにし、public な `increment` と `value` を通してのみ扱えるようにした。このように、データを隠し、使う側から使える操作だけを見せることを **カプセル化** と呼ぶ。

`count` に使う側から自由に触れられると、`Counter` は意図しない状態になりうる。たとえば、数えた覚えのない値を直接書き込まれるかもしれない。`count` を private にすれば、その変化を `increment` を通したものだけに保てる。データを隠すかどうかは、型をどう設計するかによる判断である。
