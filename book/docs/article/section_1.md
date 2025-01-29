## 1.1 Hello world (C++23)

C++23では `<print>` ライブラリが導入されたため、いきなり何もかも違います。
`std::println` は書式付きで出力してくれる関数で、出力の末尾に改行が付きます。

```cpp
#include <print>

int main() {
	std::println("{}", "Hello, C++23 world!");
}
```

## 1.3 演算子

演算子が3つ追加されている (C++20):

- <=>
- co_await
- co_yield

ひとつめは三方比較演算子（three-way comparison operator）である。
`a <=> b` による比較の結果は、未満・等しい・超える、という3つの関係を同時に表す値を返す。
戻り値の型が比較カテゴリ型になっていて、半順序・弱順序・全順序を表すことができて便利。

あとのやつはコルーチン関係だが、今は覚えなくていい。

## 1.4 条件分岐

制御構文は初期化子と条件式を分けて書ける（Since C++17）:

```cpp
if (int i = 1; i > 2) {
    // ...
}
```

制御構文には type alias が書ける（Since C++23）:

```cpp
if (using T = int; T a = 1; a > 2) {
    // ...
}
```

## 1.5 組み込み型とポインター

- 符号付き整数型が2の補数表現であることを規定 (Since C++20)
- サイズリテラル（operator ""z）（Since C++23）

## 1.7 繰り返し

range-based for は初期化子を書ける (Since C++20)

一時変数を変数に格納して range-based for を回すことができる:

```cpp
std::vector<std::string> generate() {
    return { "foo", "bar" };
}

for (std::vector vec = generate(); auto&& item : vec) {
    // ...
}
```

C++23からはそもそも範囲初期化子の一時変数は寿命が延長される:

```cpp
// itemは一時変数の要素を参照するが、C++23からはOK
for (auto&& item : generate()) {
    // ...
}
```


## 理解度チェック

1. if文の初期化子を使うメリットはなんでしょう
2. C++17で range-based for を書くときに気をつけるべき事柄はなんでしょう？
