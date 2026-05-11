---
title: "std::vector<bool>とC++20コンセプト"
emoji: "📦️"
type: "tech"
topics: ["cpp"]
published: true
---

## `std::vector<bool>` の特殊性

突然ですが`std::vector<bool>`はご存知でしょうか。
`std::vector`クラステンプレートの特殊化の一つで、`bool`をそのまま配列としてしまうと空間効率が悪いために特別な実装とすることが実装者に推奨されています。
`std::vector<bool>`の実装詳細については他に譲りますが、とにかく通常の`std::vector<T>`とはそのインターフェースが異なります。
通常`std::vector<T>::reference`は`T&`と等しいですが、`std::vector<bool>::reference`は参照のように振る舞うクラス型(しばしばプロキシ参照と呼ばれます)となっています。
これによって、`std::vector<bool>`はコンテナ要件を満たしません[^1]。
また、`std::vector<bool>::iterator`は前方向イテレータ要件すら満たしません[^2]。
これらはどちらの要件も`reference`メンバ型が真の参照型であることを要求しているからです。

## C++20未満における問題点

`<algorithm>`ヘッダで`std`名前空間直下に存在するアルゴリズムの一部は、引数に取るイテレータに対し前方向イテレータ以上の要件を課します。
`std::vector<bool>::iterator`が前方向イテレータ要件を満たさないため、そのようなアルゴリズムにこれを渡すことは未定義動作です。

## C++20におけるプロキシ参照の許容

C++20においてイテレータ要件などが刷新され、`std::vector<bool>::iterator`は`std::random_access_iterator`コンセプトのモデルとなります[^3]。
同時に、`<algorithm>`ヘッダで`std::ranges`名前空間直下に追加されたアルゴリズムは`std::vector<bool>::iterator`を正しく受け付けます。
これは、`std::forward_iterator`などのコンセプトが`reference`メンバ型がプロキシ参照を指すことを許容するように設計されているからです。

## まとめ

前方向イテレータを要求する`std`名前空間直下のアルゴリズムに`std::vector<bool>`を渡さないようにし、`std::ranges`名前空間直下のアルゴリズムを利用するようにしましょう。

[^1]: https://eel.is/c++draft/container.requirements#container.reqmts-4

[^2]: https://eel.is/c++draft/iterator.cpp17#forward.iterators-1.3

[^3]: 規格内で"A models B"とある表現を、この記事では「AはBのモデルとなる」と表現することにしています。
