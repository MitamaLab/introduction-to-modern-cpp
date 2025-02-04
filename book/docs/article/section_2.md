## 2.1 構造体・共用体・列挙体

### 2.1.1 Designated initializers (Since C++20)

aggregate class は aggregate initialization という特別な初期化ができる。
Designated initializersは、構造体やクラスのメンバを名前で指定して初期化する機能です。
C言語互換機能として導入されましたが、配列には使えないです。

以下のような特徴があります:

- メンバ名を `.メンバ名 = 初期化子` の形式で指定します
- 初期化順序は宣言順に従う必要があります
- 省略すると、デフォルト値を使うか、なければデフォルト初期化します

```cpp title="designated_init_struct.cpp"
struct product {
    int id;
    int price;
    int stock;
};

int main() {
    product pen = {
        // idはデフォルト構築されて0になる
        .price = 100,
        .stock = 200,
    };
}
```

aggregate になる条件は [cppreference](https://en.cppreference.com/w/cpp/language/aggregate_initialization) を参照してほしい。

### 2.1.2 Designated initializers (Since C++20)

Union にも使える。当たり前だが、designator はひとつしか指定できない。

```cpp title="designated_init_union.cpp"
union uni {
    int a;
    int b;
    int c;
};

int main() {
    uni u = { .b = 1 };
}

```

### 2.1.3 using enum (Since C++20)

```cpp title="using_enum.cpp"
enum class Flag {
    A,
    B,
    C,
};

int main() {
    using enum Flag;
    auto flag = A;
}
```

## 2.8 ラムダ式

### 評価されない文脈でのラムダ式が利用可能 (Since C++20)

`decltype`, `sizeof`, `noexcept`, `typeid`の引数及び、`concept`の定義内、`requires`節内の6箇所で利用できる。
特に `decltype` で使えることとデフォルト構築可能になったことで新たな使い方が可能になった。

### キャプチャしてないラムダ式はデフォルト構築可能 (Since C++20)

関数オブジェクトをテンプレート引数に受け取るクラスにデフォルト構築可能なラムダ式の型を `decltype` を使って渡せるようになった。

```cpp title="decltype_lambda_technic.cpp"
#include <iostream>
#include <set>

int main()
{
  std::set<int, decltype([](int lhs, int rhs){ return lhs > rhs;})> set{};
  set.insert({1, 2, 4, 3, 0, 10, 9, 7, 5, 6, 8, 1, 5, 10});

  for (int n : set) {
    std::cout << n << " ";
  }
  // 10 9 8 7 6 5 4 3 2 1 0
}
```

### 引数がないときの丸括弧の省略 (Since C++23)

C++20までは `[]` と `{}` の間に何も無い場合にしか省略できなかったが、なにかがあっても省略できるようになった。

```cpp
[] () static {}; // C++20
[] static {};    // C++23
```

## 理解度チェック

1. enum class を使って byte 型を作ってください
2. 最小のラムダ式を書いてみて
3. 以下の関数オーバーロードの出力結果を予想してください:
    ```cpp
    void print(const char* a) {
        std::println("const char*: {}", a);
    }

    void print(std::string a) {
        std::println("std::string: {}", a);
    }

    int main() {
        print("test");
    }
    ```
