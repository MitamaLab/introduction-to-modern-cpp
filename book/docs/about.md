## インストールが必要なもの

- ツール
    - [cabin](https://github.com/cabinpkg/cabin)
    - [clang-format](https://clang.llvm.org/docs/ClangFormat.html)
    - [clang-tidy](https://clang.llvm.org/extra/clang-tidy/)
    - [CMake](https://cmake.org/)
    - [vcpkg](https://github.com/microsoft/vcpkg)
    - llvm
    - lld
    - pkg-config
    - Git
- ライブラリ
    - [Catch2](https://github.com/catchorg/Catch2)

これらすべて Homebrew でインストールできます。

```shell
brew install cabin clang-format clang-tidy cmake llvm lld pkgconf git
```

ライブラリは vcpkg でインストールします。

```shell
vcpkg install catch2
```

## 本講義で利用するプロジェクトの構成

```
.
├── include # ヘッダライブラリを置く場所
│   └── ...
├── src     # 課題を置く場所
|   ├── CMakeLists.txt
|   └── sectoin_0.cpp
├── CMakeLists.txt 
├── Makefile.toml  # cargo-make で便利なコマンドを色々用意しました
├── cabin.toml     # cabinの設定ファイル
└── .clang-format  # clang-formatの設定ファイル、自由にいじってよい
```

