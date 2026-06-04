---
title: "値カテゴリ"
---

基本編の参照とポインタの章で、`int& r = a;` のように、名前のついたオブジェクトに別名を付ける参照を見た。式には、名前のついたオブジェクトを指すものと、その場限りの一時的な値とがある。この区別を **値カテゴリ** と呼ぶ。本章では、lvalue と rvalue という値カテゴリと、それぞれに束縛できる参照を扱う。

## lvalue と rvalue

```cpp
int a = 5;
```

`a` という式は、名前のついたオブジェクトを指す。アドレスを取れ、式が終わったあとも残る。このような式を **lvalue** と呼ぶ。

一方、`5` や `a + 1` という式は、その場で作られる一時的な値で、名前を持たない。式が終わればなくなる。このような式を **rvalue** と呼ぶ。

```cpp
a;        // lvalue。名前のついたオブジェクト
5;        // rvalue。一時的な値
a + 1;    // rvalue。計算の結果の一時的な値
```

同じ `int` の値でも、`a` のように名前で指せるものと、`a + 1` のように一時的なものとがある。値カテゴリは、この違いを表す。

## lvalue 参照と rvalue 参照

基本編で見た `int&` の参照は、lvalue にしか束縛できない。これを **lvalue 参照** と呼ぶ。

```cpp
int a = 5;
int& r = a;   // lvalue の a に束縛できる
```

一時的な値である rvalue には束縛できない。

:::message
lvalue 参照を rvalue に束縛する
```cpp
int& r = 5;
```
`5` は rvalue で、名前のついたオブジェクトではない。`int&` は lvalue にしか束縛できないので、ビルドに失敗する。
:::

rvalue に束縛するには、`int&&` と書く。これを **rvalue 参照** と呼ぶ。

```cpp
int&& r = 5;   // rvalue の 5 に束縛できる
```

なお、`const int&` は例外で、lvalue にも rvalue にも束縛できる。基本編で、コピーを避けるために `const` 参照で受け取ったのは、この性質による。

## 値カテゴリで呼び分ける

lvalue 参照と rvalue 参照は、オーバーロードの引数として使い分けられる。

```cpp
void f(int& x)  { std::println("lvalue: {}", x); }
void f(int&& x) { std::println("rvalue: {}", x); }
```

`f(int&)` は lvalue を、`f(int&&)` は rvalue を受け取る。

```cpp
int a = 5;
f(a);       // lvalue: 5
f(5);       // rvalue: 5
f(a + 1);   // rvalue: 6
```

`f(a)` では `a` が lvalue なので `f(int&)` が、`f(5)` や `f(a + 1)` では実引数が rvalue なので `f(int&&)` が呼ばれる。オーバーロード解決の章で見た候補の選び方が、値カテゴリによっても働く。

rvalue は、その場限りの一時的な値で、あとから使われることがない。そのため、`f(int&&)` が呼ばれたとき、`x` の中身を別の用途に回しても差し支えない。名前のついたオブジェクトと一時的な値を、値カテゴリはこのように区別する。

:::details 値カテゴリの分類
式は、lvalue・xvalue・prvalue の三つに分かれる。本章で rvalue と呼んだものは、このうち xvalue と prvalue をまとめた呼び名である。また、lvalue と xvalue をまとめて glvalue と呼ぶ。本章は、lvalue と rvalue の区別にとどめる。
:::
