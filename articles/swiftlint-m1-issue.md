---
title: "M1 MacでSwiftLintが動かない"
emoji: "💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Swift, SwiftLint, M1, AppleSilicon, HomeBrew]
published: true
---

## Precondition

- Apple Silicon Mac (M1)
- HomeBrew経由でSwiftLintをインストール

## Issue

> SwiftLint not installed, download from https://github.com/realm/SwiftLint

既存のプロジェクトをM1 Macでビルドした際、警告が出る事がある。

多くのプロジェクトでは下記のようなスクリプトを追加して、ビルド時に自動で解析できるように設定するかと思う。
当該の事象は `swiftlint` のpathが上手く取得できずにelse側の処理が実行された結果となる。

```sh
if which swiftlint >/dev/null; then
    swiftlint
else
    echo "warning: SwiftLint not installed, download from https://github.com/realm/SwiftLint"
fi
```

## Answer

これはHomeBrewでインストールする際のpathが、Intel版とM1版で異なることに起因する。
M1版でのpathをXcodeが取得できない状態となっているので、下記のようにスクリプトを修正する必要がある。

```sh
if test -d /opt/homebrew/bin; then
    PATH=/opt/homebrew/bin/:$PATH
fi

if which swiftlint >/dev/null; then
    swiftlint
else
    echo "warning: SwiftLint not installed, download from https://github.com/realm/SwiftLint"
fi
```


## Reference

https://github.com/realm/SwiftLint/issues/2992
