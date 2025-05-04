## 型推論

参照型の推論をする場合、`auto&` を使うことができます。
この場合は、参照型であることは絶対です。

```cpp
auto& x = 0; // これはエラー
```

実は、`decltype(auto)` というものもあります。
これを使うと、初期化子の型全体を推論できます。
参照を返す関数は参照型として推論されますし、値を返す関数は値型として推論されます。

```cpp
int& id(int& i) {
    return i;
}
int main() {
    decltype(auto) x = 0; // これは int 型
    decltype(auto) y = x; // これも int 型
    decltype(auto) z = id(x); // これは int& 型
    decltype(auto) _ = (x);   // これも int& 型
}
```

## ムーブセマンティクスと右辺値参照

C++11 からはムーブセマンティクスが導入されました。

TODO

## 理解度チェック

1. TODO

2. TODO

3. TODO
