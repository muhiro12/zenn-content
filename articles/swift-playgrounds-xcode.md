---
title: "XcodeとPlaygroundsを共存させるプロジェクト構成"
emoji: "🕊️"
type: "tech"
topics: [Swift, Playgrounds, Xcode, iPad, plist]
published: false
---

## はじめに

***iPadでiOSアプリを開発する***
これはiOSエンジニアとしてのロマンの一つだと思います。

**Swift Playgrounds** の機能も増えてきており、iPadで開発を完結させることも不可能ではなくなってきました。
実は `Capabilities` などの設定もPlaygroundsで行えるため、本当に様々な機能を持ったアプリを作ることができます。

ただ残念なことに全ての `Capabilities` を設定できるわけではありません。
例えば `iCloud` 関連の設定はできないようになっています。

他にも

- Info.plistで詳細な設定を行いたい
- Run Scriptを設定したい
- Xcode Cloudと連動させたい

など、Xcodeなら簡単に出来ることが、Playgroundsでは難しい場合があります。

そこで、本記事ではXcodeとPlaygroundsを共存させるプロジェクト構成を紹介します。

## 環境

- Xcode 15.4
- Swift Playgrounds 4.5
- iOS 17.5
- Xcode Cloud

全てが必須条件ではないですが、制約が厳しくない状況下という点が伝われば嬉しいです。

## 手順

### Xcodeプロジェクトの作成

1. Xcodeプロジェクト `Sample` を作成します。
2. `Hello, world` の代わりに `Xcode` と表示するように変更します。
3. Xcodeプロジェクトを実行して `Xcode` が表示されることを確認します。

### Playgroundsプロジェクトの作成

1. Playgroundsプロジェクト `Sample` を作成します。
2. `Hello, world` の代わりに `Playgrounds` と表示するように変更します。
3. Playgroundsプロジェクトを実行して `Playgrounds` が表示されることを確認します。

ここで改めてとなりますが、
**Xcodeプロジェクトを実行した時に `Playgrounds` が表示される**
が今回実現したいこととなります。

### Xcodeワークスペースの作成

1. Xcodeで新規ワークスペース `Sample` を作成します。
2. サイドバーに `Sample.xcodeproj` をドラッグ&ドロップします。

### パッケージの作成

1. ワークスペースを開いた状態で、空のパッケージ `MyLibrary` を追加します。
2. `MyLibrary` の配下に `Sample.swiftpm` を配置します。
3. `Sample.swiftpm` 内に `Sources` ディレクトリを作成し、`ContentView.swift` を `Sources` 配下に移動します。
4. `MyLibrary` の `Package.swift` を設定します。

```swift
let package = Package()
```

ポイントは **3** で `Package.swift` , `SampleApp.swift` 以外のファイルを別ディレクトリに移動することです。
ディレクトリ名は `Sources` 以外でも問題ありません。

`Package.swift` , `SampleApp.swift` の2ファイルを含んだ状態だとビルドが通らなくなります。
これを防ぐため、 `MyLibrary` のパスを `Sources` に設定し、2ファイルをパッケージから除外しています。

### パッケージのインポート

1. frameworkとして設置します。
2. `import MyLibrary` します。
3. Playgrounds側の `ContentView` を設定します。
4. Xcodeプロジェクトを実行して `Playgrounds` が表示されることを確認します。
