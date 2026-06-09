---
title: "ranges とアルゴリズム"
---

前章までで、コンテナは `begin`/`end` とイテレータで巡れることを見た。要素を並べ替える、探す、数えるといった操作は、コンテナを巡るイテレータの上で書ける。標準ライブラリは、こうした操作を **アルゴリズム** として用意している。使うには `<algorithm>` を取り込む。本章では、C++20 で入った `std::ranges` のアルゴリズムを主に扱う。これは範囲(コンテナのように `begin`/`end` で巡れるもの)を直接受け取る。条件や基準は、言語編のラムダの章で見たラムダで渡す。

## 並べ替える

`std::ranges::sort` は、範囲の要素を並べ替える。

```cpp
std::vector<int> v = {5, 3, 1, 4, 2};
std::ranges::sort(v);
for (int x : v) std::println("{}", x);   // 1 2 3 4 5
```

`std::ranges::sort(v)` は、`v` の要素を昇順に並べ替える。並べ替えの基準を変えるには、二つの要素の順序を決める比較をラムダで渡す。

```cpp
std::ranges::sort(v, [](int a, int b) { return a > b; });
for (int x : v) std::println("{}", x);   // 5 4 3 2 1
```

ラムダ `[](int a, int b) { return a > b; }` は、`a` を `b` より前に置くかを返す。`a > b` を返すので、大きいものが前に来て、降順になる。

`std::ranges::sort` は要素を並べ替えるので、要素を書き換えられる範囲にしか使えない。

:::message
並べ替えられない範囲を sort する
```cpp
std::set<int> s = {3, 1, 2};
std::ranges::sort(s);
```
`std::set` の要素は、連想コンテナの章で見たとおり、イテレータ越しに書き換えられない。並べ替えは要素を書き換えるので、`std::ranges::sort` は `std::set` を受け取れず、ビルドに失敗する。
:::

## 探す

`std::ranges::find` は、与えた値に等しい最初の要素を指すイテレータを返す。見つからなければ `end()` を返す。

```cpp
std::vector<int> v = {5, 3, 1, 4, 2};
auto it = std::ranges::find(v, 4);
if (it != v.end()) std::println("{}", *it);   // 4
```

値そのものでなく、条件で探すには `std::ranges::find_if` にラムダを渡す。ラムダは、要素が条件を満たすかを返す。

```cpp
auto it = std::ranges::find_if(v, [](int x) { return x % 2 == 0; });
if (it != v.end()) std::println("{}", *it);   // 4
```

`{5, 3, 1, 4, 2}` の最初の偶数は `4` なので、`find_if` はその `4` を指すイテレータを返す。

`find` や `find_if` の戻り値は、使う前に `end()` と比べる。`end()` は末尾の次を指し、要素を指さない。確かめずに取り出すと、要素でない場所を読む。

:::message alert
見つからなかった結果を確かめずに取り出す
```cpp
std::vector<int> v = {1, 2, 3};
auto it = std::ranges::find(v, 99);
std::println("{}", *it);
```
`99` は `v` にないので、`find` は `end()` を返す。`end()` は要素を指さないのに、`*it` でその先を取り出している。ビルドは通るが、結果は定まらず、実行時に深刻な問題が起こりうる。
:::

## 数える

`std::ranges::count_if` は、条件を満たす要素の数を返す。

```cpp
std::vector<int> v = {5, 3, 1, 4, 2};
auto n = std::ranges::count_if(v, [](int x) { return x % 2 == 0; });
std::println("{}", n);   // 2
```

`{5, 3, 1, 4, 2}` の偶数は `4` と `2` の二つなので、`2` を返す。

## 一部分で並べ替え・探す

これまでの `sort` や `find` は、要素そのものを比べた。要素から取り出した一部分で並べ替えたり、探したり、数えたりしたいこともある。`std::ranges` のアルゴリズムは、比べる前に各要素へ適用する関数を取れる。これを **projection** と呼ぶ。

名前と年齢を持つ `Person` の並びで見る。

```cpp
struct Person
{
    std::string name;
    int age;
};
```

年齢で並べ替えるには、各要素から `age` を取り出す projection を渡す。

```cpp
std::vector<Person> people = {{"bob", 30}, {"alice", 25}, {"carol", 40}};
std::ranges::sort(people, {}, [](Person const& p) { return p.age; });
// age の昇順に並ぶ: alice(25), bob(30), carol(40)
```

`sort` の三つめの引数が projection で、各要素に先に適用される。projection は三つめなので、二つめの比較を省けない。既定の昇順でよいときは、二つめに `{}` を置いて既定の比較を表す。`sort` は、projection が返した `age` どうしを比べて並べ替える。

メンバを取り出すだけなら、`&Person::age` と短く書ける。これは `Person` の `age` メンバを指すもので、projection として渡すと、各要素の `age` を取り出す。

```cpp
std::ranges::sort(people, {}, &Person::age);   // 同じく age の昇順
```

projection は `find` や `count_if` でも使える。このとき、要素そのものでなく、projection が返した値が、比べられたりラムダに渡されたりする。

```cpp
auto it = std::ranges::find(people, "carol", &Person::name);            // name が "carol" の要素
auto n = std::ranges::count_if(people, [](int a) { return a >= 30; }, &Person::age);   // age が 30 以上の数
```

`find` は各要素の `name` を `"carol"` と比べる。`count_if` のラムダには、projection が返した `age` が渡るので、仮引数は `age` と同じ `int` になる。projection によって、要素そのものでなく、その一部分で並べ替えたり、探したり、数えたりできる。

:::details イテレータとセンチネルの組を取る形
`std::ranges::sort` は、範囲のほかに、言語編で見たイテレータとセンチネルの組も受け取れる。

```cpp
std::ranges::sort(v.begin(), v.end());        // 範囲全体
std::ranges::sort(v.begin(), v.begin() + 3);  // 先頭の3つだけ
```

`begin()` から `end()` までを渡せば範囲全体を、途中までを渡せばその一部だけを並べ替えられる。範囲の一部だけを対象にしたいときは、この組を取る形を使う。

`std::ranges` より前からある `std::sort` は、範囲を取らず、このイテレータの組だけを取る。範囲を直接渡せるのは、`std::ranges` のアルゴリズムである。

```cpp
std::sort(v.begin(), v.end());
```
:::

`std::ranges` のアルゴリズムは、コンテナを巡って並べ替えたり探したりする操作だった。次章では、範囲から別の範囲を導く、ビューとレンジアダプタを見ていく。
