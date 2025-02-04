## Explicitly-defaulted definition (Since C++11)

特殊メンバ関数の `default` 指定を使えば、`inline`/`virtual` 修飾をしながら自動実装を利用できます。
特に `virtual` 修飾する場合のデストラクタなどは (RAII管理外のヒープ領域等がなければ) 特に書くこともないはずなので `default` 指定するのが望ましいです。

```cpp title="defaulted.cpp"
class foo {
    int value;
public:
    foo(int value): value{value} {} // user-provided definition
    virtual ~foo() = default; // explicitly-defaulted definition
}
```

## Decucing this (Since C++23)

C++23 からは this ポインタを経由する以外にも、明示的に自分自身を受け取る書き方ができるようになりました。

```cpp title="deducing_this.cpp"
struct X {
    void foo(this X const& self, int i);
};

int main() {
    X x;
    x.foo(42);      // 'self' is bound to 'x', 'i' is 42
}
```

次のようなメンバ関数は同じシグネチャとなるので、エラーとなります。

```cpp
struct X {
    void foo() const&;
    void foo(this X const& self); // error!
};
```

??? "ラムダ式とDeducing this"
    ラムダ式は匿名クラスなので、Deducing this が使えます。
    ラムダ式がラムダ式自身を受け取れるということなので、当然再帰的ラムダ式が書けます。

    ```cpp
    #include <utility>

    inline constexpr auto factorial = [](this auto&& self, int n) -> int {
        return n == 0 ? 1 : n * std::forward_like<decltype(self)>(self)(n - 1);
    };

    int main() {
        factorial(5); // 120
    }
    ```

??? warning "Deducing this の真の力"
    Deducing this はテンプレートを使ったときにすべての力を開放できます。
    テンプレートを用いると、const性や参照も含めて推論されるため、オーバーロードする必要がありません。
    cv-qualifier によるオーバーロードが全く違う処理をすることはほぼナイでしょうから、
    実践的には使うのはほぼ次のような見た目をした関数になるでしょう……。
    （このコードを理解するには _Reference Collapsing_ と _Perfect Forwarding_ を知る必要があります、これらは9章のテンプレートで扱われます）

    ```cpp
    #include <utility>

    class X {
        int value_ = 0;
    public:
        auto&& value(this auto&& self) {
            return std::forward_like<decltype(self)>(self).value_;
        }
    };

    int main() {
        X x;
        x.value() = 1; // ok

        const X cx;
        cx.value() = 1; // error 
    }
    ```

## Inline static data members

C++17 から static data member が `inline` 指定できるようになったので、`inline` 指定しましょう。
`inline` 指定すると、外部リンケージを持つようになります。つまり、複数の翻訳単位 (ソースファイル) から同一の定義を参照するようになります。

ちなみに、`constexpr` static data member の場合は暗黙に `inline` 指定されます (定数なので)。

```cpp title="inline_static.hpp"
#include <atomic>
// ヘッダに直接定義をすることができる
struct X {
    static inline std::atomic<int> count = {};
};
```

## 継承はあまり使わないという話

!!! warning
    C++はパフォーマンスを重視するタイプの言語なので、パフォーマンスが落ちる可能性がある継承の使い方をあまりしません。
    C++ではインターフェイスのような概念を扱う場合は基本的にコンセプト (Since C++20) を用います。

    AnimalクラスがあってCatとDogが云々みたいなのはC++ではやらないです、
    どうしても抽象化して同じコンテナに入れたい時があり、その場合は `std::variant` を使うことが多いのではないかと思います。

    Rustでいうと、`dyn Trait` を使う場合は、`std::error::Error` を扱うときですよね。C++ でもそういう場合は `std::shared_ptr` を使う必要があります。

## 理解度チェック

1. 次のクラスに `value()` メンバ関数を deducing this を使って実装してください:
   ```cpp
   class Counter {
       int count_ = 0;
   public:
       // ここに実装
   };
   ```

2. TODO

3. TODO

