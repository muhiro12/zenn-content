---
title: "CoreDataからSwiftDataへデータの引き継ぎ"
emoji: "💾"
type: "tech"
topics: [Swift, SwiftData, CoreData, iOS]
published: true
---

## はじめに

現在、CoreDataを利用したアプリをリリースしています。
SwiftDataの発表を受け、移行作業を進めています。

その際 **CoreData版で保存したデータがSwiftData版に引き継がれない** という問題に直面しました。

その解決方法の一つをこの記事に記します。


## 前提

**Adopting SwiftData for a Core Data app**

https://developer.apple.com/documentation/coredata/adopting_swiftdata_for_a_core_data_app

この記事では上記のドキュメントおよびサンプルコードを基本として話を展開します。
Appleから提供されたコードではデータの引き継ぎができなかったため、サンプルコードに手を加える形で解決方法を説明します。

## 結論

CoreData版でデータを保存した場所のURLを指定することで、SwiftData版にデータ引き継ぎすることができます。

```swift
import SwiftUI
import SwiftData

@main
struct TripsApp: App {
    let container = {
        let url = URL.applicationSupportDirectory.appendingPathComponent("Trips.sqlite")
        let configuration = ModelConfiguration(url: url)
        return try! ModelContainer(for: Trip.self, configurations: configuration)
    }()

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(container)
    }
}
```

## TripsApp（サンプルコード）について

Appleのサンプルコードとして、3つのXcodeプロジェクトが提供されています。
BundleIDは同じなので、全て同じアプリとして認識されます。

- **Trips-CoreData** - CoreDataのみを利用するプロジェクト
- **Trips-Coexistence** - アプリ本体ではCoreData、ウィジェットではSwiftDataを利用するプロジェクト
- **Trips-SwiftData** - SwiftDataのみを利用するプロジェクト

旅行の計画 `Trip` を作成して確認することができる、というアプリです。

## 動作確認

下記の手順でアプリの起動、データの作成を行います。

1. **Trips-CoreData** プロジェクトを実行
2. `Trip` を作成
3. **Trips-SwiftData** プロジェクトを実行
4. 2で作成した `Trip` を確認

1はアプリの新規インストール、3は新しいバージョンへのアップデート、と考えることができます。
この時、4では `Trip` が表示されることを期待しますが、実際は何も表示されません。

CoreData版で保存したデータがSwiftData版に引き継がれていない、ということです。
どうすれば4で `Trips` が表示されるのか、を考えていきます。

## SwiftDataではどこにデータが保存されるのか

> Additionally, the app sets up the container using ModelContainer to ensure that all views access the same ModelContainer.

```swift
@main
struct TripsApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: Trip.self)
    }
}
```

ドキュメント [*](https://developer.apple.com/documentation/coredata/adopting_swiftdata_for_a_core_data_app#4295880) およびサンプルコードでは、`ModelContainer` を設定することで全ての画面から同じデータにアクセスすることができる、と記載されています
では、そのコンテナはどこにあるのか？どこを見ているのか？という疑問が生まれます。

`ModelContainer` [*](https://developer.apple.com/documentation/swiftdata/modelcontainer) の中身を見ると、

```swift
public class ModelContainer : Equatable, @unchecked Sendable {
    final public let schema: Schema
    final public let migrationPlan: (SchemaMigrationPlan.Type)?
    public var configurations: Set<ModelConfiguration>
    // ...
}
```

さらに、怪しそうな `ModelConfiguration` [*](https://developer.apple.com/documentation/swiftdata/modelconfiguration) の中身を見ると、

```swift
public struct ModelConfiguration : Identifiable, Hashable {
    public let url: URL
    public let name: String
    // ...
}
```

となっています。
`ModelConfiguration.url` が関係していそうです。
デフォルトでどのようなURLが設定されるのか気になるので `print` で確認してみます。

```swift
let modelConfiguration = ModelConfiguration()
let url = modelConfiguration.url
print(url)
```

出力結果：

```zh
file:///.../Containers/Data/Application/.../Library/Application%20Support/default.store
```

どうやら `default.store` というファイルにデータが保存されるようです。


## CoreDataではどこにデータが保存されるのか

CoreData版サンプルコードでは `NSPersistentContainer` からURLを取得することができます。

```swift
let persistentStoreDescription = NSPersistentContainer(name: "Trips").persistentStoreDescriptions.first!
let url = persistentStoreDescription.url!
print(url)
```

出力結果：

```zh
file://.../Containers/Data/Application/.../Library/Application%20Support/Trips.sqlite
```

`Trips.sqlite` というファイルに保存されていることが分かります。

特別な実装をしていなければ、その他の多くのアプリでも同じ場所に保存されているはずです。
この場所の取得方法は下記の通り。

```swift
let url = URL.applicationSupportDirectory.appendingPathComponent("Trips.sqlite")
```

## 解決方法

結論としては `default.store` ではなく `Trips.sqlite` に保存するよう指定すれば良い、ということになります。

SwiftData版サンプルコードでは `modelContainer(for:inMemory:isAutosaveEnabled:isUndoEnabled:onSetup:)` [*](https://developer.apple.com/documentation/swiftui/scene/modelcontainer(for:inmemory:isautosaveenabled:isundoenabled:onsetup:)-82y49) を使って、間接的にコンテナを設定しています。

```swift
@main
struct TripsApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: Trip.self)
    }
}
```

今回の場合、URLを指定したいのでコンテナを自作して `modelContainer(_:)` [*](https://developer.apple.com/documentation/swiftui/scene/modelcontainer(_:)) から直接コンテナを設定します。

```swift
@main
struct TripsApp: App {
    let container = {
        let url = URL.applicationSupportDirectory.appendingPathComponent("Trips.sqlite")
        let configuration = ModelConfiguration(url: url)
        return try! ModelContainer(for: Trip.self, configurations: configuration)
    }()

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(container)
    }
}
```

SwiftData版サンプルコードへの変更は以上です。

> 下記の手順でアプリの起動、データの作成を行います。
> 
> 1. **Trips-CoreData** プロジェクトを実行
> 2. `Trip` を作成
> 3. **Trips-SwiftData** プロジェクトを実行
> 4. 2で作成した `Trip` を確認

3のプロジェクトに手を加えた状態で同様の手順を試すと、4で `Trip` が表示されることを確認できるはずです。

## English Title

Migrating Data from CoreData to SwiftData