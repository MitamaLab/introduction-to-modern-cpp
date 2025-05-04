## C++20の新しいキーワード

C++20 ではたくさんのキーワードが増えました:

- concept (Since C++20) `[テンプレート]`
- requires (Since C++20) `[テンプレート]`
- consteval (Since C++20) `[コンパイル時計算]`
- constinit (Since C++20) `[コンパイル時初期化]`
- co_await (C++20) `[コルーチン]`
- co_return (C++20) `[コルーチン]`
- co_yield (C++20) `[コルーチン]`
- char8_t (C++20) `[UTF8の型]`

文脈依存キーワード:

- import (C++20) `[モジュール]`
- module (C++20) `[モジュール]`

## Module

モジュールは C++20 から導入されたんですが、コンパイラがそれぞれ勝手に実装しているので、互換性とか考慮されてなくて最高です。
コンパイラによって実装状況とか挙動とか拡張子とか全然違います。
実用レベルとは言えないため、説明を割愛します。

## inline static メンバ変数

C++14 まではヘッダでは static 変数の定義を行えないです。
これは複数のソースファイルからインクルードされた場合に複数の実体ができて ODR 違反になってしまうからですよね。

```cpp
// ヘッダ
struct X {
  // ヘッダでは変数の宣言のみを行いソースファイルで定義する
  static int foo;
};
```

C++17 からはヘッダでも定義が行えるようになったんです、inline 変数のおかげですよ。

```cpp
struct Counter {
    inline static int count = 0;
};
```

詳しい仕様は [cpprefjp](https://cpprefjp.github.io/lang/cpp17/inline_variables.html) を参照のこと。

## constinit キーワード

C++20 からは constinit というキーワードが追加されました。
constinit はコンパイル時に初期化されることを保証するキーワードです。
主に、static 変数に使います。
static 変数の動的初期化順序がどうなるのかはほとんど不定であるため、初期化機序を明確にするために constinit が導入されたという経緯があります。

constinit は変数の初期化に使います（初期化を伴っていない場合は使えないということです）。

```cpp
constinit int x = 0; // コンパイル時に初期化されることを保証する
```

`constinit` は `constexpr` と似ていますが、`constinit` はコンパイル時に初期化されることを保証するだけで、`constexpr` はコンパイル時に評価されることを保証します。

`constinit` 指定された変数の初期化子は `constexpr` でなければなりません。
初期化子の式はコンパイル時に評価される必要があるため、部分式はに使用される変数・関数・コンストラクタは `constexpr` でなければなりません。

```cpp
struct Counter {
  constinit inline static int count = 0;
};
```

## クラススコープ

クラススコープはクラスが作るスコープです。
このスコープは特別で、宣言した名前をどこからでもアクセスできます。

```cpp
class my_class {
public:
  auto my_func() -> int {
    return value; // value は my_func よりあとに宣言されているがアクセス可能
  }
private:
  int value = 0;
};
```

## inline namespace

inline 指定された名前空間は名前空間の修飾を省略できます。
もちろん修飾つきで呼ぶこともできます。

inline namespace は上級者向けの機能です。
使いみちは主に以下の2つです:

- ライブラリ等での API のバージョン管理
- using namespaceによる名前空間省略の階層の段階的管理

ライブラリ等での API のバージョン管理

inline namespace を使うことによってデフォルトの関数を切り替えるという使い方です。

```cpp
# include <iostream>

namespace my_lib {
  namespace v1 {
    void f()
    {
      std::cout << "v1" << std::endl;
    }
  }

  inline namespace v2 {
    void f()
    {
      std::cout << "v2" << std::endl;
    }
  }
}

int main()
{
  my_lib::v1::f(); // 古いバージョンのAPIを呼び出す
  my_lib::v2::f(); // バージョンを明示的に指定して新しいAPIを呼び出す
  my_lib::f();     // ライブラリデフォルトのAPI（新しいAPI）を呼び出す
}
```

using namespaceによる名前空間省略の階層の段階的管理

おそらく UDLs (User Defined Literals) を書くときに一番使います。
なぜなら、UDLs は using directive を前提としているからです。

```cpp
namespace my_literals {
inline namespace integer_literals {
inline auto operator"" _kilo(unsigned long long x)
  { return int(x) * 1'000; }
}
inline namespace floating_point_literals {
inline auto operator"" _kilo(long double x)
  { return double(x) * 1'000; }
}
}

int main() {
    {
        using namespace my_literals::integer_literals; // int用だけ可視化される
        1_kilo; // 1000
    }

    {
        using namespace my_literals::floating_point_literals; // double用だけ可視化される
        1_kilo; // 1000.0
    }

    {
        using namespace my_literals; // 全部可視化される
        1_kilo; // 1000
        1.0_kilo; // 1000.0
    }
}
```

## 理解度チェック

1. static 変数をヘッダファイルで定義するときはどう書けばいいか。

2. constinit とは何か。
   どんなときに使うべきか。
   どんなときに使わないか。

3. 次の関数 `my_func` を v2 にマイグレーションしたい。名前空間を使って新しい関数（単に "v2" を出力する同名関数）を定義し、元の関数を非推奨にせよ。

```cpp
#include <print>

namespace my_lib {
  void my_func()
  {
    std::println("{}", "v1");
  }
}
```
