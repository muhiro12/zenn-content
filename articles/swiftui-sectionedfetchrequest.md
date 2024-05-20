---
title: "SwiftUI × CoreData でセクション分けされたリストを作る"
emoji: "🎬"
type: "tech"
topics: [Swift, SwiftUI, CoreData]
published: true
---

## やりたいこと

特定の要素でグルーピングしてセクション付きリストを作成したい。

![](/images/swiftui-sectionedfetchrequest/1.png =300x)

## つかうもの

**SectionedFetchRequest**
https://developer.apple.com/documentation/SwiftUI/SectionedFetchRequest

## 条件

iOS 15.0+

## サンプル^[説明のため簡略化したコードとなります]

```swift:ContentView.swift
struct ContentView: View {
    @SectionedFetchRequest(
        // グルーピングに利用する要素を指定
        sectionIdentifier: \Item.category,
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
                        Text(item.name) // りんご
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

## 注意点^[関連：https://developer.apple.com/forums/thread/694609]

`sortDescriptors` で `name` のみを指定した場合、グルーピングが上手く働かないことがあります。
そのような場合はグルーピングしたい要素（ `category` ）でのソートを優先するように明示することで解決します。
この点についてはドキュメントにも記載されています。

> Be sure that you choose sorting and sectioning that work together to avoid discontiguous sections.

https://developer.apple.com/documentation/swiftui/sectionedfetchrequest

推測となりますが、データの取得およびレンダリングのタイミングによる問題だと思われます。

例えば、名前でソートして1000件のデータを取得するようなケースで、1件目と1000件目にカテゴリ「ドリンク」のデータが存在するとします。この状態でドリンクのセクションを表示する場合、1~1000の全てのデータの同時取得およびグルーピングが必要となりますが現実的には難しいかと思われます。
実際は遅延取得が行われ、1件目と1000件目ではグルーピングや画面表示のタイミングがズレてしまう形となり想定する動きとなりません。このような問題を防ぐためにあらかじめカテゴリでソートしておけば、連続したデータとして取得できタイミングも揃えることができるという話だと思います。

## まとめ

これまでは取得した一覧データをDictionaryの [`init(grouping:by:)`](https://developer.apple.com/documentation/swift/dictionary/3127163-init) などを使ってグルーピングする前処理が必要でしたが、iOS15からはSwiftUIが勝手にやってくれるのでスッキリ書くことができるようになりました。