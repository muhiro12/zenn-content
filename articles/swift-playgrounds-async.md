---
title: "SwiftPlaygroundsでasync/awaitを使う"
emoji: "🌲"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Swift"]
published: true
---

## はじめに

SwiftPlaygroundsで非同期処理( `async/await` )を実装した場合に上手く動かないことがあります。

## 問題

`start` から5秒後に `end` と出力するコードについて考えます。
単純に

```swift
Task {
    print("start")
    try! await Task.sleep(nanoseconds: 5_000_000_000)
    print("end")
}
```

と実装した場合、

```
start
```

と出力され `end` が確認できません。

## 解決策

`PlaygroundSupport` をimportし

```swift
import PlaygroundSupport

PlaygroundPage.current.needsIndefiniteExecution = true

Task {
    print("start")
    try! await Task.sleep(nanoseconds: 5_000_000_000)
    print("end")
}
```

と実装することで

```
start
end
```

と期待通りの結果が出力されます。

## needsIndefiniteExecution

https://developer.apple.com/documentation/playgroundsupport/playgroundpage/1964501-needsindefiniteexecution

> By default, all top-level code is executed, and then execution is terminated. When working with asynchronous code, enable indefinite execution to allow execution to continue after the end of the playground’s top-level code is reached.

基本的にPlaygroundsでコードを **実行** した場合、最後の行に達した時点で完了したと見做され実行終了します。
しかし非同期処理の場合は、最後の行に達した後にも待機していた処理が呼び出される可能性があります。

`needsIndefiniteExecution` を `true` にすることで **無期限の実行** を可能にします。
これにより実行したコードが終了せずに、非同期処理の結果を確認することができるようになります。

### 実行終了

言葉の通り **無期限** なので `true` にしたままだと、永遠に実行終了することはありません。
実行終了のタイミングをPlaygroundsに伝えてあげることで、より丁寧なコードになります。

```swift
import PlaygroundSupport

PlaygroundPage.current.needsIndefiniteExecution = true

Task {
    print("start")
    try! await Task.sleep(nanoseconds: 5_000_000_000)
    print("end")
    PlaygroundPage.current.needsIndefiniteExecution = false
}
```