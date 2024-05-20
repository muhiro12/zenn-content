---
title: "XcodeとPlaygroundsを共存させるプロジェクト構成"
emoji: "🕊️"
type: "tech"
topics: [Swift, Playgrounds, Xcode, iPad, plist]
published: false
---

## はじめに

***iPadでのiOSアプリ開発***
これはiOSエンジニアとしてロマンの一つだと思います。

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

||||
|-|-|-|
|![](/images/swift-playgrounds-xcode/1.png)|![](/images/swift-playgrounds-xcode/2.png)|![](/images/swift-playgrounds-xcode/3.png)|

2. `Hello, world!` の代わりに `Xcode` と表示するように変更します。
3. Xcodeプロジェクトを実行して `Xcode` が表示されることを確認します。
4. Xcodeプロジェクトを閉じます。

### Playgroundsプロジェクトの作成

1. Playgroundsプロジェクト `Sample` を作成します。

||||
|-|-|-|
|![](/images/swift-playgrounds-xcode/1.png)|![](/images/swift-playgrounds-xcode/4.png)|![](/images/swift-playgrounds-xcode/5.png)|

2. `Hello, world` の代わりに `Playgrounds` と表示するように変更します。
3. Playgroundsプロジェクトを実行して `Playgrounds` が表示されることを確認します。
4. Playgroundsプロジェクトを閉じます。

ここで改めてとなりますが、
**Xcodeプロジェクトを実行した時に `Playgrounds` が表示される**
が今回実現したいこととなります。

### Xcodeワークスペースの作成

1. Xcodeでワークスペース `Sample` を作成します。
2. 空のサイドバーに `Sample.xcodeproj` をドラッグ&ドロップします。

||||
|-|-|-|
|![](/images/swift-playgrounds-xcode/6.png)|![](/images/swift-playgrounds-xcode/7.png)|![](/images/swift-playgrounds-xcode/8.png)|

### パッケージの作成

1. 空のパッケージ `MyPackage` を作成し、`Add to` , `Group` を設定します。

||||
|-|-|-|
|![](/images/swift-playgrounds-xcode/9.png)|![](/images/swift-playgrounds-xcode/10.png)|![](/images/swift-playgrounds-xcode/11.png)|

2. `MyPackage` の配下に `Sample.swiftpm` を配置します。
3. `Sample.swiftpm` 配下に `Sources` ディレクトリを作成し、`ContentView.swift` を `Sources` に移動します。
4. `MyPackage` の `Package.swift` を設定します。

```swift
// swift-tools-version: 5.10
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
    name: "MyPackage",
    platforms: [.iOS(.v17)],
    products: [.library(name: "MyPackage", targets: ["MyPackage"])],
    targets: [.target(name: "MyPackage", path: "Sample.swiftpm/Sources")]
)
```

ポイントは **3** で `Package.swift` , `SampleApp.swift` 以外のファイルを別ディレクトリに移動することです。

`Package.swift` , `SampleApp.swift` の2ファイルを含んだ状態だとビルドが通らなくなります。
これを防ぐため `MyPackage` のパスを `Sources` に設定し、2ファイルをパッケージから除外しています。

### パッケージのインポート

1. 作成した `MyPackage` をXcodeプロジェクトに組み込みます。

||||
|-|-|-|
|![](/images/swift-playgrounds-xcode/12.png)|![](/images/swift-playgrounds-xcode/13.png)|![](/images/swift-playgrounds-xcode/14.png)|

2. Playgrounds側の `ContentView` のアクセスレベルを変更します。

```swift
import SwiftUI
```

3. Xcode側の `ContentView.swift` を削除します。
4. Xcode側の `SampleApp`　を修正します

```swift
import SwiftUI
```

5. Xcodeプロジェクトを実行して `Playgrounds` が表示されることを確認します。

## 運用方法

### 開発時

普段の開発時 つまり Swiftを書く場合は `Sample.swiftpm` を開いて作業します。
Xcode、Playgroundsどちらで開いても問題ありません。

### 設定変更時

プロジェクトの設定を変更したい場合は `Sample.xcworkspace` を開いて作業します。

- Info.plistの設定
- Run Scriptの設定
- Xcode Cloudの設定

などがXcodeのGUI上で出来るようになります。

## 注意点

`SampleApp.swift` と **プロジェクトの設定ファイル** がそれぞれ2つずつ存在する形になります。
その点に注意して運用する必要があります。

### `SampleApp.swift`

出来る限りAppでは **何もせずにContentViewを呼び出すだけ** の方針することをオススメします。
何かの処理が必要な場合もXcode, Playgroundsで同じ形になるようにしましょう。

### プロジェクトの設定ファイル

**Xcode CloudでArchiveする** 方針にすることをオススメします。
PlaygroundsでArchiveすると、Xcode側で設定した内容が反映されない形でリリースすることになります。
今回の構成を活かすためには **Xcode** か **Xcode Cloud** でのArchiveが必要です。

iPadでの開発を叶えるためには **Xcode Cloud** 一択になりますよね！

## こだわる人向けに

***Coming Soon***

## おわりに

Playgroundsプロジェクトのコードをパッケージ化し、Xcodeプロジェクトから利用する方針で構成しました。
