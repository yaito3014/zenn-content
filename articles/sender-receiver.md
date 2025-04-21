---
title: "C++26 の std::execution の概念まとめ [WIP]"
emoji: "💨"
type: "tech"
topics:
  - "cpp"
published: true
---

# 概要

[P2300 `std::execution`](https://wg21.link/p2300) がついに C++26 入りしたとのことで、自分のためにもどういった設計になっているのかを用語を整理しながら見ていこうと思います。

# CPO の種別について

`std::execution` には多くの CPO (customization point object) が登場しますが、どうやらそれらには種別があるようです(規格書にも下記と同様の表が載っています)

| 種別                 | 説明                                            | 具体例                                                       |
| -------------------- | ----------------------------------------------- | ------------------------------------------------------------ |
| core                 | `std::execution` の機能のコアとなる             | `start`, `connect`                                           |
| completion functions | sender が完了を通知する際に呼ばれる関数       | `set_value`, `set_error`, `set_stopped`                      |
| senders              | sender アルゴリズムのカスタマイズを可能にする | `schedule`, `then`, `sync_wait`                              |
| queries              | オブジェクトへの性質の問い合わせを可能にする    | `get_allocator`, `get_scheduler`, `get_completion_scheduler` |

# 用語の定義など

## *query object*, *queryable object*
*queryable object* は *query object* と呼ばれる CPO を Key とする Key/Value ペアの集合であり、*query object* に *queryable object* を渡すことで各 *query object* に対応する *queryable object* の性質を問い合わせることができます。

## *execution resource*
*execution resource* は並列で処理を行う可能性のある *execution agent* を管理するプログラムの実体であり、*asynchronous operation* を実行します。

## *asynchronous operation*
*asynchronous operation* は結果のデータと次の三つの完了状態(*disposition*)のうちいずれかを以て完了するような独立した処理です。
- 成功 (successful completion a.k.a. *value completion*)
  - 任意個の結果のデータを持つことができる
- 失敗 (failure completion a.k.a. *error completion*)
  - 一つの結果のデータを持つ
- キャンセル (cancellation completion a.k.a. *stopped completion*)
  - 結果のデータを持たない

また、開始されたのと異なる *execution resource* において完了したり、自身を *parent operation* として、自身が完了するより前に完了する *child operation* を開始する可能性があります。

## *operation state*, *environment*, *receiver*
*asynchronous operation* には *operation state* と呼ばれる関連した状態があり、*environment* という呼び出し元の実行時の性質を表す *queryable object* と *receiver* を所有します。
*asynchronous operation* には関連した *receiver* があり、これは完了状態に応じた三つのハンドラを集約したものです。
*receiver* に関連する *environment* は *operation state* に関連する *environment* と等しいです。

## *completion function*, *completion signature*
*asynchronous operation* の三つの完了状態のそれぞれに対し *completion function* と呼ばれる CPO が存在し、*receiver* と結果のデータを引数に取り *receiver* のハンドラを呼び出します。
正当(valid)な *completion function* の呼び出しは *completion operation* と呼ばれます。
*completion signature* は *completion operation* を表す関数の型です。

## *sender*, *attribute*
*sender* は一つ以上の *asynchronous operation* のためのファクトリであり、 *sender* と *receiver* とを `std::execution::connect` によって *connect* することで *asynchronous operation* が作成されます。
*sender* には関連する *queryable object* があり、*attribute* と呼ばれます。

## *scheduler*
*scheduler* は *exection resource* の抽象化であり、処理のスケジューリングに対する汎用なインターフェースを提供します。
*scheduler* は *sender* に対するファクトリであり、 *scheduler* `sch` に対する `std::execution::schedule(sch)` の呼び出しにより *sender* が作成されます。

## *completion scheduler*
*asynchronous operation* には一つ以上の関連した *completion scheduler* があり、それらは *completion operation* を実行する *scheduler* です。

## sender algorithm
sender アルゴリズムは *sender* を受け渡しする関数群であり、三つのカテゴリに分類されます。
- *sender factory*
  - 非 *sender* を引数に取り *sender* を返す
- *sender adaptor*
  - *sender* と(あれば)追加の引数を取り *sender* を返す
- *sender consumer*
  - *sender* を引数に取り非 *sender* を返す

# 詳細

## *query object* の一覧

### `std::forwarding_query`
`std::forwarding_query` は *query object* 自体に問い合わせを行う *query object* です。その仕様は「*query object* `q` に対して `std::forwarding_query(q)` が `false` を返す場合、*queryable object* `env` と *exposition-only entity* `FWD-ENV(...)` について `FWD-ENV(env).query(q, args...)` が *ill-formed* になる」というものです。
つまり、特定の箇所において `env` を引き渡す際に引き渡し以降の問い合わせを禁止するかどうかというもののようです。

```cpp
struct my_query_object_t {
  constexpr bool query(std::forwarding_query_t) { return false; }

  template <class Env>
  decltype(auto) operator()(const Env& env) const noexcept {
    return env.query(my_query_object_t{});
  }
};

inline constexpr my_query_object_t my_query_object{};

struct my_receiver {
  using receiver_concept = std::execution::receiver_t;
  void set_value(auto&&...) const noexcept {}
  void set_error(auto&&) const noexcept {}
  void set_stopped() const noexcept {}
  struct env {
    std::allocator<void> query(std::get_allocator_t) const noexcept { return {}; }
    int query(my_query_object_t) const noexcept { return 42; }
  };
  env get_env() const noexcept { return {}; }
};

static_assert(    std::forwarding_query(std::get_allocator));
static_assert(not std::forwarding_query(   my_query_object));

my_receiver recv;
auto env = std::execution::get_env(recv);

auto alloc = std::get_allocator(env);  // well-formed
auto my_value = my_query_object(env);  // well-formed

auto env2 = FWD-ENV(env);  // FWD-ENV を通す

auto alloc2 = std::get_allocator(env2);  // well-formed
auto my_value2 = my_query_object(env2);  // !!! ill-formed !!!
```

### `std::get_allocator`

*queryable object* に関連する *allocator* を問い合わせます。例えば *asynchronous operation* を表すオブジェクトが特定の *allocator* を使用して構築されている場合それを使用して破棄する必要があり、そういった場面で使われるのだと思われます。

### `std::get_stop_token`

*queryable object* に関連する stop token を問い合わせます。

### `std::execution::get_env` 

*sender* に対しては *attribute* を、*receiver* に対しては *environment* を問い合わせます。
元々は前者の問い合わせのために `std::execution::get_attr` なる *query object* が存在していましたが、`std::execution::get_env` に統合されたようです。

### `std::execution::get_domain`

*domain* と呼ばれる、*sender* が完了する *scheduler* に関連するタグ型を問い合わせます。

### `std::execution::get_scheduler`

*queryable object* に関連する *scheduler* を問い合わせます。

### `std::execution::get_delegation_scheduler`

*queryable object* に関連する、forward progress delegation に適した *scheduler* を問い合わせます。

### `std::execution​::​get_forward_progress_guarantee`

*scheduler* に対し、その *scheduler* に関連する *execution resouce* の作る *execution agents* の forward progress guarantee を問い合わせます。


#### `std::execution::forward_progress_guarantee`

それぞれの forward progress guarantee を表す enum class です。

```cpp
enum class forward_progress_guarantee{
  concurrent,
  parallel,
  weakly_parallel,
};
```

### `std::execution::​get_completion_scheduler`

*sender* の *attribute* から *completion scheduler* を問い合わせます。

## scheduler

`std::execution::scheduler` concept は *scheduler* の型に対する要求を定義します。
具体的には以下の通りです。

```cpp
namespace std::execution {

template<class Sch>
concept scheduler =
    derived_from<typename remove_cvref_t<Sch>::scheduler_concept, scheduler_t> &&
    queryable<Sch> &&
    requires(Sch&& sch) {
      { schedule(std::forward<Sch>(sch)) } -> sender;
      { auto(get_completion_scheduler<set_value_t>(get_env(schedule(std::forward<Sch>(sch))))) } -> same_as<remove_cvref_t<Sch>>;
    } &&
    equality_comparable<remove_cvref_t<Sch>> &&
    copyable<remove_cvref_t<Sch>>;

}
```

ここで `std::execution::schedule` は *scheduler* を受け取る customization point object であり、適格な `schestd::execution::schedule` の呼び出しは *schedule-expression* と呼ばれます。
コピーや比較の操作が例外によって終了してはならず、`schedule` の呼び出しも含めて複数のスレッドから呼び出されたとしてもデータ競合を起こしてはいけません。
同じ *execution resouce* を共有している場合にのみ比較関数は `true` を返すべきであり、`get_completion_scheduler<set_value_t>(get_env(schedule(sch)))` は `sch` と等しい必要があります。

## receiver

*receiver* は *asynchronous operation* の継続を表します。
`std::execution::receiver` concept は *receiver* の型に対する要求を定義します。
具体的には以下のとおりです。

```cpp
namespace std::execution {

template<class Rcvr>
concept receiver =
  derived_from<typename remove_cvref_t<Rcvr>::receiver_concept, receiver_t> &&
  requires(const remove_cvref_t<Rcvr>& rcvr) {
    { get_env(rcvr) } -> queryable;
  } &&
  move_constructible<remove_cvref_t<Rcvr>> &&
  constructible_from<remove_cvref_t<Rcvr>, Rcvr>;

}
```

`final` 指定されたクラス型は `std::execution::receiver` を表しません。

### `std::execution::set_value`

*value completion function* であり、 関連する completion tag は `std::execution::set_value_t` それ自身です。

### `std::execution::set_error`

*error completion function* であり、関連する completion tag は `std::execution::set_error_t` それ自身です。

### `std::execution::set_stopped`

*stopped completion function* であり、関連する completion tag は `std::execution::set_stopped_t` それ自身です。

---

🏗️工事中
