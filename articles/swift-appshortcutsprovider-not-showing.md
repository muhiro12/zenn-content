---
title: "AppShortcutsProviderを実装してもショートカットに表示されない"
emoji: "🙈"
type: "tech"
topics: [Swift, iOS, AppIntents, AppShortcutsProvider, Siri]
published: true
---

## はじめに

Apple Intelligenceの発表に伴い、App Intentsへの関心も高まっているはずです。
AppShortcutsProviderを使えば、作成したApp Intentsをショートカットとしてユーザーに提供することが可能となります。

しかし、AppShortcutsProviderを実装しても、ショートカットアプリに表示されないことがあります。
本記事では表示されない場合の解決策について記します。

## 環境

- Xcode 16.1 beta
- iOS 18.1 beta

## 解決策

### フレーズに `.applicationName` を含める

AppShortcutのフレーズにアプリ名が含まれていないと、ショートカットとして認識されない場合があります。
フレーズに `.applicationName` を含めるようにしましょう。

#### ⭕️ 表示される

```swift
static var appShortcuts: [AppShortcut] = [
    AppShortcut(
        intent: YourIntent(),
        phrases: [
            "Do something in \(.applicationName)"
        ],
        shortTitle: "Your Shortcut",
        systemImageName: "star"
    )
]
```

#### ❌ 表示されない

```swift
static var appShortcuts: [AppShortcut] = [
    AppShortcut(
        intent: YourIntent(),
        phrases: [
            "Do something"
        ],
        shortTitle: "Your Shortcut",
        systemImageName: "star"
    )
]
```

### `.init()`ではなく`AppShortcut()`を使う

AppShortcutのインスタンス生成時 `.init()` で初期化するとショートカットとして認識されない場合があります。
明示的に `AppShortcut()` でインスタンス生成するようにしましょう。

#### ⭕️ 表示される

```swift
static var appShortcuts: [AppShortcut] = [
    AppShortcut(
        intent: YourIntent(),
        phrases: [
            "Do something in \(.applicationName)"
        ],
        shortTitle: "Your Shortcut",
        systemImageName: "star"
    )
]
```

#### ❌ 表示されない

```swift
static var appShortcuts: [AppShortcut] = [
    .init(
        intent: YourIntent(),
        phrases: [
            "Do something in \(.applicationName)"
        ],
        shortTitle: "Your Shortcut",
        systemImageName: "star"
    )
]
```

### `updateAppShortcutParameters()` を呼び出す

> To register the App Shortcut with the system, the app calls updateAppShortcutParameters on its AppShortcutsProvider during the init of the App structure.

Appleの[サンプルコード](https://developer.apple.com/documentation/appintents/acceleratingappinteractionswithappintents#Create-App-Shortcuts)では `updateAppShortcutParameters` を呼び出すように記載されています。
Appの初期化時にこちらを呼び出すようにしましょう。

:::details 備考
私の手元では呼び出さない場合でもショートカットアプリに反映されました。
念の為、呼び出しておく方が無難だと思います。
:::

```swift
@main
struct YourApp: App {
    init() {
        YourAppShortcutsProvider.updateAppShortcutParameters()
    }
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

## おわりに

App Intentsはこれから盛んになっていく機能だと思います。
今後、細かな挙動やドキュメント類も整備されていくでしょう。

取り急ぎ、Xcode 16 beta時点の挙動として残しておきます。
