## メンバー関数のオーバーロードと非メンバー関数のオーバーロード

演算子のオーバーロードには、メンバー関数として定義する方法と非メンバー関数として定義する方法の2つがあります。
`operator->`/`operator()`/`operator[]`/`operator=` の4つはメンバー関数として定義する必要があります。

メンバ関数として定義すると、第一引数は必ず自身のクラスのオブジェクトになります。
対して、非メンバー関数として定義すると、第一引数は自身のクラスのオブジェクトである必要はありません。
よって、テンプレートを使って汎用的に定義することができます。
ライブラリを設計する場合は、非メンバー関数として定義することが多いです。

## multidimensional subscript operator (Since C++23)

C++23 からは `operator[]` を多次元配列用にオーバーロードできるようになりました。
`operator[]` の引数を複数受け取ることができます。

```cpp
#include <array>
#include <cassert>
#include <iostream>
 
template<typename T, std::size_t Z, std::size_t Y, std::size_t X>
struct Array3d
{
    std::array<T, X * Y * Z> m{};
 
    constexpr T& operator[](std::size_t z, std::size_t y, std::size_t x) // C++23
    {
        assert(x < X and y < Y and z < Z);
        return m[z * Y * X + y * X + x];
    }
};
 
int main()
{
    Array3d<int, 4, 3, 2> v;
    v[3, 2, 1] = 42;
    std::cout << "v[3, 2, 1] = " << v[3, 2, 1] << '\n';
}
```

`v[3, 2, 1]` のように、 `operator[]` に複数の引数を渡すことができます。
このコードは C++20 まではカンマ演算子として解釈されてしまい、意図した動作をしません。
C++20からは `operator[]` に複数の引数を渡すことは規格で非推奨に指定されています。
C++20以前の挙動を期待する場合は、`v[(3, 2, 1)]` のように、引数を丸括弧で囲めばできます。

## 演算子のオーバーロードと explicit object parameter (C++23)

C++23 からは、メンバー関数として定義する演算子のオーバーロードにおいても explicit object parameter (3章を参照) を使うことができるようになりました。
これはまだ利点がわからないかもしれないですが、テンプレートを使うときに威力を発揮します。

```cpp
#include <iostream>

struct Test
{
	int value;

	constexpr Test operator+(this auto const& self, Test const& other)
	{
		return Test{ .value = self.value + other.value };
	}
};

int main()
{
	Test a{ 1 };
	Test b{ 2 };
	Test c = a + b;
	std::cout << c.value << '\n';
}
```

## static overloaded operators (Since C++23)

C++23からは this ポインタを持つ必要がない静的メンバー関数として `operator()` と `operator[]` をオーバーロードできるようになりました。
this ポインタが存在しないため、explicit object parameter を使うこともできません。

### static call/subscript operator

```cpp
struct SwapThem
{
    template<typename T>
    static void operator()(T& lhs, T& rhs) 
    {
        std::ranges::swap(lhs, rhs);
    }

    template<typename T>
    static void operator[](T& lhs, T& rhs)
    {
        std::ranges::swap(lhs, rhs);
    } 
};
inline constexpr SwapThem swap_them{};
 
void foo()
{
    int a = 1, b = 2;
 
    swap_them(a, b); // OK
    swap_them[a, b]; // OK
 
    SwapThem{}(a, b); // OK
    SwapThem{}[a, b]; // OK
 
    SwapThem::operator()(a, b); // OK
    SwapThem::operator[](a, b); // OK
 
    SwapThem(a, b); // error, invalid construction
    SwapThem[a, b]; // error
}
```

### static lambda

同様にラムダ式も匿名クラスであるため、静的メンバー関数として `operator()` をオーバーロードできます。
ラムダキャプチャが空のラムダ式であれば、`static` をつけることができます。
当然ですが this ポインタは存在しませんので `mutable` 指定はできません、もちろん explicit object parameter も使えません。

```cpp
[]() static {}
//   ^^^^^^ ここ
```

## 三方比較演算子 `operator<=>` (Since C++20)

### 三方比較演算子とは

C++20 からは三方比較演算子 `operator<=>` が導入されました。
三方比較演算子は、2つの値を比較して、等しいか、左側が小さいか、右側が大きいかを返します。
これを使うことで、すべての比較演算子の機能を1つの関数で実装できます。
そして、この演算子を定義するだけで、`==`、`!=`、`<`、`<=`、`>`、`>=` の6つの比較演算子が自動的に生成されます。

C++17までの比較演算子のオーバーロードは、すべての比較演算子を個別に定義する必要がありました。

```cpp
#include <tuple>

struct S  {
  int x;
  double d;
  char str[4];

  constexpr bool operator<(const S& rhs) const {
    return std::tie(x, d, str[0], str[1], str[2], str[3])
         < std::tie(rhs.x, rhs.d, rhs.str[0], rhs.str[1], rhs.str[2], rhs.str[3]);
  }

  constexpr bool operator==(const S& rhs) const {
    return std::tie(x, d, str[0], str[1], str[2], str[3])
        == std::tie(rhs.x, rhs.d, rhs.str[0], rhs.str[1], rhs.str[2], rhs.str[3]);
  }

  constexpr bool operator!=(const S& rhs) const {
    return !(*this == rhs);
  }

  constexpr bool operator>(const S& rhs) const {
    return rhs < *this;
  }

  constexpr bool operator<=(const S& rhs) const {
    return !(*this > rhs);
  }

  constexpr bool operator>=(const S& rhs) const {
    return !(*this < rhs);
  }
};
```

C++20 からは、三方比較演算子を定義するだけで、すべての比較演算子が自動的に生成されます。

```cpp
#include <compare>

struct S {
  int x;
  double d;
  char str[4];

  auto operator<=>(const S&) const = default;
};
```

### Explicitly defaulted 定義と delete 定義

特殊メンバ関数と比較演算子は、`default` 指定と `delete` 指定を使うことができます。
特殊メンバ関数とは、デフォルトコンストラクタ、コピーコンストラクタ、ムーブコンストラクタ、コピー代入演算子、ムーブ代入演算子、デストラクタのことです。
比較演算子とは、`operator==`、`operator<=>`、`operator!=` (ただし、`operator==` が明示的に定義されている場合のみ)、`operator<` (ただし、`operator<=>` が明示的に定義されている場合のみ)、`operator<=` (ただし、`operator<=>` が明示的に定義されている場合のみ)、`operator>` (ただし、`operator<=>` が明示的に定義されている場合のみ)、`operator>=`  (ただし、`operator<=>` が明示的に定義されている場合のみ) のことです。

まず、`delete` 指定について説明します。
`delete` 指定を使うと、その関数の呼び出しをコンパイルエラーにすることができます。
これは、すべての関数やオペレータに対して適用できます。
例えば、コピーコンストラクタを `delete` 指定すると、オブジェクトのコピーが禁止されます。

```cpp
struct NoCopy
{
    NoCopy(const NoCopy&) = delete;
    NoCopy& operator=(const NoCopy&) = delete;
};
```

つぎに、`default` 指定について説明します。
`default` 指定を使うと、その関数のデフォルト実装 (自明な実装) を生成することができます。
自明な実装とはコンパイラにとって自明な実装という意味です、つまり効率的な実装が生成されます。

### 三方比較演算子の `default` 指定

三方比較演算子に `default` 指定を使うと、その型のすべてのメンバを (上から順に) 辞書式順序で比較する三方比較演算子が生成されます。
すべてのメンバが三方比較演算子をサポートしている必要があります。
しかし C++17 までに書かれたコードではすべてのクラス三方比較演算子をサポートしていないため、追加のルールがあります。
`operator==` と `operator<` の両方が定義されている場合は、その定義を使って `operator<=>` を自動的に合成します。


## 理解度チェック

1. `static` 指定されたメンバー関数としての演算子オーバーロードとそうでないものを [Compiler Explorer](https://godbolt.org/) で比較してみましょう。

2. 

3. 三方比較演算子を定義することで、どのような利点がありますか？

