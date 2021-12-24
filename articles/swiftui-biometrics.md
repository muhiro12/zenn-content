---
title: "SwiftUI時代のアプリロック（生体認証）"
emoji: "🔑"
type: "tech"
topics: ["Swift", "SwiftUI", "FaceID", "生体認証"]
published: false
---

## はじめに

アプリに鍵をかけたい、という要件が発生することがたまにあるかと思います。
SwiftUIを採用している場合は比較的簡単に達成できる機能ではありますが、async/awaitの到来でよりスマートに実装できるようになりました。

## 前提

- Swift 5.5^

## サンプル

```swift
import SwiftUI
import LocalAuthentication

struct ContentView: View {
    @State var isUnlocked = false

    var body: some View {
        Group {
            if isUnlocked {
                HomeView()
            } else {
                // ユーザーによるキャンセル等が発生した場合を考えてボタンを残しておく
                Button("ロック解除") {
                    unlockWithBiometrics()
                }
            }
        }.onAppear {
            unlockWithBiometrics()
        }
    }

    func unlockWithBiometrics() {
        Task {
            isUnlocked = await doBiometrics()
        }
    }

    // お好みで別クラスへの切り出しも可能
    func doBiometrics() async -> Bool {
        let context = LAContext()
        do {
            return try await context.evaluatePolicy(.deviceOwnerAuthentication,
                                                    localizedReason: "ロック解除")
        } catch {
            return false
        }
    }
}
```

生体認証周りのAPIは既にConcurrencyに対応しているため、 **async/await** および **do/catch** の形で書くことができます。

生体認証の結果を戻り値としてBoolで返すことができるので、簡単に別のクラスに切り出すこともできます。
クロージャとは異なり `error` がオプショナルではないため、その点でも取り回しが良いです。


## 参考

https://developer.apple.com/documentation/localauthentication/lacontext/1514176-evaluatepolicy