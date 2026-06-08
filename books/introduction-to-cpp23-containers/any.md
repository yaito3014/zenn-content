---
title: "任意の型"
---

`std::variant` は、あらかじめ挙げた型のうち一つを持った。型を挙げずに、どんな型でも持てるようにしたいこともある。それが `std::any` である。

## 任意の型を持つ

`std::any` は、任意の型の値を一つ持つ。使うには `<any>` を取り込む。

```cpp
std::any a = 5;
a = std::string("hi");
```

`a` は、初め `int` の `5` を持ち、後で `std::string` を持つ。

## 中身を取り出す

`std::any_cast<T>` で、`T` の値を取り出す。

```cpp
std::any a = 5;
std::println("{}", std::any_cast<int>(a));   // 5
```

`std::any` は、持っている値の型を覚えていて、`std::any_cast` は、取り出す型がそれと合うかを確かめる。合わない型で取り出すと、`std::bad_any_cast` を投げる。

```cpp
std::any a = 5;                  // int を持つ
std::any_cast<std::string>(a);   // std::bad_any_cast を投げる
```

何も持たない `std::any` もある。持っているかは `has_value` で調べる。

```cpp
std::any empty;
std::println("{}", empty.has_value());   // false
```

## 閉じた型と開いた型

`std::variant` と `std::any` は、どちらも一つの値を、その型とともに持つ。違いは、持てる型の範囲である。`std::variant` は、あらかじめ挙げた型に限る、閉じた集合である。`std::any` は、型を挙げず、どんな型でも持てる、開いた集合である。挙げた型だけでよいなら `std::variant`、どんな型でも持てるようにしたいなら `std::any` を使う。

どんな型の値でも、その型を表に出さずに一様に持つしくみを **型消去** と呼ぶ。`std::any` では、持っている型が実行時まで保たれ、`std::any_cast` が取り出しのたびに、その型と合うかを確かめる。
