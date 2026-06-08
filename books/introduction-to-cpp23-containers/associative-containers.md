---
title: "連想コンテナ"
---

前章の `std::vector` と `std::array` は、同じ型の値を並べて持ち、要素を位置(添字)で取り出した。値を、位置でなく **キー** で引きたいこともある。名前から年齢を引く、といった対応づけである。それを担うのが **連想コンテナ** で、本章では `std::map`・`std::set`、順序を保たない `std::unordered_map`・`std::unordered_set`、そして要素を連続して持つ `std::flat_map`・`std::flat_set`(C++23)を扱う。

## キーで引く

`std::map<K, V>` は、キー `K` から値 `V` を引くコンテナである。使うには `<map>` を取り込む。

```cpp
std::map<std::string, int> ages = {{"alice", 30}, {"bob", 25}};
ages["carol"] = 28;
std::println("{}", ages["alice"]);   // 30
for (auto const& [name, age] : ages)
    std::println("{}: {}", name, age);
```

`std::map<std::string, int>` は、`std::string` のキーから `int` の値を引く。`ages["alice"]` は、キー `"alice"` に結びついた値を返す。`ages["carol"] = 28` は、キー `"carol"` に値 `28` を結びつける。

`std::map` を範囲for で巡ると、要素はキーと値の組として取り出せる。言語編の構造化束縛の章で見た `[name, age]` で、その組をキーと値に分ける。`std::map` は要素をキーの順に並べて保つので、巡る順はキーの昇順になる。

`[]` には、注意するところがある。`[]` は、キーがなければ、そのキーを値とともに挿入してから、その値への参照を返す。挿入される値は、基本編のデフォルト初期化で見た、値の定まらない場合とは違い、数値型なら `0` に定まる。読むだけのつもりで存在しないキーに `[]` を使うと、要素が増える。

```cpp
std::map<std::string, int> ages = {{"alice", 30}};
std::println("{}", ages["dave"]);   // 0    "dave" が値 0 とともに挿入される
std::println("{}", ages.size());    // 2    要素が増えている
```

挿入せずに調べたいときは、別の手立てを使う。キーがあるかは `contains` が返す。キーに結びついた値は `at` で取り出せて、`at` はキーがなければ `std::out_of_range` を投げる。これは、文字列の章で見た `at` と同じである。

```cpp
std::map<std::string, int> ages = {{"alice", 30}};
ages.contains("alice");   // true
ages.contains("zoe");     // false
ages.at("zoe");           // std::out_of_range を投げる
```

## 重複しない要素

`std::set<T>` は、重複しない `T` の要素を持つコンテナである。使うには `<set>` を取り込む。

```cpp
std::set<int> s = {3, 1, 4, 1, 5};
s.insert(9);
for (int x : s) std::println("{}", x);   // 1 3 4 5 9
std::println("{}", s.contains(4));        // true
```

同じ値を二度入れても、`std::set` は一つだけ持つ。初期化子の `1` は二つあるが、残るのは一つである。`std::set` も要素を順に並べて保つので、巡ると昇順になる。`std::map` がキーから値を引いたのに対し、`std::set` はキーだけを持ち、そのキーがあるかを `contains` で問う。

`std::set` の要素は書き換えられない。要素は順に並べて保たれており、要素そのものを書き換えると、その並びの前提が崩れる。そのため、イテレータ越しに要素を書き換えることはできない。

:::message
set の要素をイテレータ越しに書き換える
```cpp
std::set<int> s = {1, 2, 3};
auto it = s.begin();
*it = 10;
```
`std::set` の要素は書き換えられないので、`*it` への代入はできない。ビルドに失敗する。
:::

## 消した要素を指すイテレータ

`std::map` と `std::set` は、要素を一つずつ別の場所に置いて保つ。`erase` で要素を消すと、その要素の置かれていた場所は解放される。前章の `std::vector` で見た無効化と同じく、消された要素を指していたイテレータや参照は、解放された場所を指したままになる。

:::message alert
消した要素を指すイテレータを使う
```cpp
std::map<std::string, int> ages = {{"alice", 30}, {"bob", 25}};
auto it = ages.find("alice");
ages.erase("alice");
std::println("{}", it->second);
```
`it` は `"alice"` の要素を指す。`erase("alice")` でその要素が消され、置かれていた場所は解放される。`it` は解放された場所を指したままになる。ビルドは通るが、`it` を使うと、結果は定まらず、実行時に深刻な問題が起こりうる。
:::

ただし、無効化の及ぶ範囲は `std::vector` と違う。`std::vector` の再確保は、要素をまとめて別の領域へ移すため、それまでのイテレータや参照を一斉に無効にしうる。`std::map` と `std::set` の `erase` が無効にするのは、消した要素を指すものだけで、他の要素を指すイテレータや参照は有効なまま残る。要素を一つずつ別の場所に置く保ち方の違いによる。

## 順序を保たないコンテナ

`std::map` と `std::set` は、要素をキーの順に並べて保った。順序を保たず、ハッシュに基づいて要素を格納するコンテナもある。`std::unordered_map` と `std::unordered_set` である。使うには `<unordered_map>`・`<unordered_set>` を取り込む。

```cpp
std::unordered_map<std::string, int> ages = {{"alice", 30}, {"bob", 25}};
ages["carol"] = 28;
std::println("{}", ages["alice"]);   // 30
```

`[]`・`at`・`find`・`contains`・`insert` は、`std::map`・`std::set` と同じように使える。違うのは要素の並びで、`std::map` がキーを順に並べて保つのに対し、`std::unordered_map` は要素を巡る順を定めない。

`std::unordered_map` は、キーから **ハッシュ** と呼ばれる値を求め、それで要素の格納場所を振り分ける。キーから格納場所を直接求められるので、多くの場合、要素数が増えても引く手間は大きく変わらない。`std::map` は、キーを順に並べて保つぶん、引くときにその並びをたどり、要素数が増えると引く手間も増える。順序が要るなら `std::map`・`std::set` を、順序が要らず引く速さを重視するなら `std::unordered_map`・`std::unordered_set` を使う。規格では、前者を連想コンテナ、後者を非順序連想コンテナと呼んで区別する。

キーをハッシュで振り分け、同じ格納場所に集まったキーを見分けるには、キーの型がハッシュを求められ、かつ等しいかを比べられる必要がある。`int` や `std::string` のような標準の型は、そのまま使える。自分で定義した型をキーにするには、その型のためのハッシュと、等しいかの比較を用意する必要がある。

:::message
ハッシュのない型をキーにする
```cpp
struct Point { int x; int y; };
std::unordered_set<Point> s;
```
`Point` には、ハッシュも、等しいかの比較も用意されていない。`std::unordered_set` のキーにできず、ビルドに失敗する。
:::

## 連続して持つ連想コンテナ

C++23 では、要素を連続した領域に並べて持つ連想コンテナ、`std::flat_map` と `std::flat_set` が加わった。使うには `<flat_map>`・`<flat_set>` を取り込む。

```cpp
std::flat_map<std::string, int> ages = {{"bob", 25}, {"alice", 30}};
ages["carol"] = 28;
for (auto const& [name, age] : ages)
    std::println("{}: {}", name, age);
// alice: 30 / bob: 25 / carol: 28  (キーの順)
```

`std::flat_map` は `std::map` と同じく、`[]`・`at`・`find`・`contains` で引け、要素をキーの順に並べて保つ。違うのは、要素の持ち方である。`std::map` が要素を一つずつ別の場所に置いたのに対し、`std::flat_map` は、キーの並びと値の並びを、それぞれ連続した領域に、キーの順に並べて持つ。`std::flat_set` も同じく、`std::set` の要素を一つの連続した領域に持つ。

これらは、内部に `std::vector` のような連続したコンテナ(`std::flat_map` ならキー用と値用の二つ)を持ち、それを連想コンテナとして見せる。このような型を **コンテナアダプタ** と呼ぶ。要素を連続して並べて持つので、たどるときは連続した領域を読む。一方、連続して並べて持つぶん、途中に要素を挿入したり消したりすると、後ろの要素をずらすことになる。

:::details SoA と AoS
`std::flat_map` のように、キーの配列と値の配列に分けて持つ持ち方を **SoA**(structure of arrays)と呼ぶ。キーと値を組にして、その組を一列に並べる持ち方は **AoS**(array of structures)である。`std::flat_map` は、`pair` の組を一列に並べる(AoS)のでなく、キーと値をそれぞれの配列に分ける(SoA)。SoA では、キーを探すときに、キーの配列だけをたどればよい。
:::

ここまでの連想コンテナ(順序を保つ `std::map`・`std::set`、ハッシュの `std::unordered_map`・`std::unordered_set`、連続して持つ `std::flat_map`・`std::flat_set`)は、キーから値を引いたり、キーがあるかを問うたりするコンテナだった。次章では、コンテナの要素を巡って探したり変えたりする操作、`std::ranges` のアルゴリズムを見ていく。
