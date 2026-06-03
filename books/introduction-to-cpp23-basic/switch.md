---
title: "switch"
---

条件分岐の章では、`if` と `else` で、条件が成り立つかによって分岐した。一つの値が、いくつもの場合のどれに当たるかで分けたいとき、`if`/`else` を連ねると長くなる。**`switch`** は、一つの値を、その値ごとに用意した場合へ振り分ける。

## switch 文

整数 `n` の値で分ける。

```cpp
int n = 2;
switch (n)
{
case 1:
    std::println("one");
    break;
case 2:
    std::println("two");
    break;
default:
    std::println("other");
    break;
}
```

`switch (n)` は、`n` の値を見て、それに一致する `case` へ進む。`n` は `2` なので `case 2:` に進み、`two` を出力する。どの `case` にも一致しないときは、`default:` に進む。`case` の処理の終わりには `break` を書く。この `break` は、break と continue の章で見たものと同じで、ここでは `switch` を抜ける。

`switch` で分けられるのは整数だけでなく、列挙型の値でもよい。列挙型 `Color` の値なら、`case Color::red:` のように書いて分けられる。

## break を忘れると次の case に流れ落ちる

`case` の処理に `break` を書かないと、その処理を終えたあと、`switch` を抜けずに、次の `case` の処理へそのまま続けて進む。

```cpp
int n = 1;
switch (n)
{
case 1:
    std::println("one");
case 2:
    std::println("two");
    break;
}
```

`case 1:` に `break` がないため、`one` を出力したあと、そのまま `case 2:` の処理に進み、`two` も出力する。`n` は `1` なのに、`one` と `two` の両方が出る。`break` を書かずに次の `case` へ流れ落ちることは、意図して用いる場合もある。ただ、多くは書き忘れによるもので、それでもビルドは通るため、実行するまで現れない。
