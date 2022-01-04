---
title: "SwiftUI x CoreData でセクション分けされたリストを作る"
emoji: "🎬"
type: "tech"
topics: [Swift, SwiftUI, CoreData]
published: false
---

## やりたいこと

特定の要素でグルーピングしてセクション付きリストを作成したい。

![](/images/swiftui-sectionedfetchrequest-1.png =300x)

## つかうもの

**SectionedFetchRequest**
https://developer.apple.com/documentation/SwiftUI/SectionedFetchRequest

## サンプル^[説明のため簡略化したコードとなります]

```swift:ContentView.swift
struct ContentView: View {
    @SectionedFetchRequest(
        // グルーピングに利用する要素を指定
        sectionIdentifier: \Item.category!,
        // ソートの指定
        sortDescriptors: [NSSortDescriptor(keyPath: \Item.category, ascending: true),
                          NSSortDescriptor(keyPath: \Item.name, ascending: true)],
        animation: .default)
    private var sections: SectionedFetchResults<String, Item>

    var body: some View {
        List {
            ForEach(sections) { section in
                Section(content: {
                    ForEach(section) { item in
                        Text(item.name!) // りんご
                    }
                }, header: {
                    // `section.id` で指定した要素(カテゴリ)を取得可能
                    Text(section.id) // フルーツ
                })
            }
        }
    }
}
```

```swift:Item.swift
@objc(Item)
public class Item: NSManagedObject {}

extension Item: Identifiable {
    @NSManaged public var category: String
    @NSManaged public var name: String
    @NSManaged public var price: NSDecimalNumber
}
```

## 注意点

`sortDescriptors` で `name` のみを指定した場合、

https://developer.apple.com/forums/thread/694609?answerId=700281022#700281022