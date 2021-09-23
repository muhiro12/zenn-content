---
title: "BottomNavigationBarItemをハンドリングする"
emoji: "🤹"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "RiverPod", "Dart", "iOS", "Android"]
published: true
---

## タブタップ時に特定の処理を行いたい

BottomNavigationBarItem(以下 タブ)をタップした際に何かの処理が必要なシーンはよくある。

- 画面のリロード
- 最上部までスクロール
- 最初の画面まで戻す

Riverpod `1.0.0-dev.7` を使って、これらを叶える方法を考える。

## タブ管理の基本構造

選択中のタブをChangeNotifierProviderで管理する形とする。
ProviderをConsumerWidgetで講読し、状態に応じて表示するタブのインデックスを更新する。
またタブがタップされた際は `BottomTab.select` メソッド経由で状態を更新する。

最新の値のみではなく、一つ前の値 `oldItem` を定義することで、ハンドリングの自由度を高める。

```dart:bottom_tab.dart
final ChangeNotifierProvider<BottomTab> bottomTabProvider =
    ChangeNotifierProvider<BottomTab>((_) => BottomTab());

class BottomTab extends ChangeNotifier {
  BottomTabItem _item = BottomTabItem.home;
  BottomTabItem _oldItem = BottomTabItem.home;

  BottomTabItem get item => _item;
  BottomTabItem get oldItem => _oldItem;

  void select(BottomTabItem newItem) {
    _oldItem = _item;
    _item = newItem;
    notifyListeners();
  }
}

enum BottomTabItem {
  home,
  scroll,
  navigation,
}
```

```dart:app.dart
List<BottomTabItem> get _items => BottomTabItem.values;

@override
Widget build(BuildContext context, WidgetRef ref) {
  final BottomTab bottomTab = ref.watch(bottomTabProvider);
  final int currentIndex = _items.indexOf(bottomTab.item);
  return MaterialApp(
    home: Scaffold(
      body: IndexedStack(
        index: currentIndex,
        children: ...
      ),
      bottomNavigationBar: BottomNavigationBar(
        onTap: (int index) => bottomTab.select(_items[index]),
        currentIndex: currentIndex,
        items: ...
      ),
    ),
  );
}
```

## タブタップのハンドリング

`ref.listen` (旧ProviderListener)でProviderの変更を講読する。
一つ前の状態を取得できるため、各タブ内で自由にハンドリングできる。

```dart:home_page.dart
@override
Widget build(BuildContext context, WidgetRef ref) {
  ref.listen(
    bottomTabProvider,
    (BottomTab bottomTab) {
      if (bottomTab.item != BottomTabItem.home ||
          bottomTab.item != bottomTab.oldItem) {
        return;
      }
      // ホーム画面表示中にホームタブをタップした場合
    },
  );
  return Scaffold(
    ...
  );
}
```

## サンプル

:::details サンプルコード

### 画面のリロード

```dart:home_page.dart
ref.listen(
  bottomTabProvider,
  (BottomTab bottomTab) {
    if (bottomTab.item != BottomTabItem.home ||
        bottomTab.item != bottomTab.oldItem) {
      return;
    }
    ref.refresh(provider);
  },
);
```

### 最上部までスクロール

```dart:scroll_page.dart
ref.listen(
  bottomTabProvider,
  (BottomTab bottomTab) {
    if (bottomTab.item != BottomTabItem.scroll ||
        bottomTab.item != bottomTab.oldItem) {
      return;
    }
    scrollController.animateTo(
      0,
      duration: const Duration(milliseconds: 300),
      curve: Curves.linear,
    );
  },
);
```

### 最初の画面まで戻す

```dart:navigation_page.dart
ref.listen(
  bottomTabProvider,
  (BottomTab bottomTab) {
    if (bottomTab.item != BottomTabItem.navigation ||
        bottomTab.item != bottomTab.oldItem) {
      return;
    }
    Navigator.of(context).popUntil((Route<dynamic> route) => route.isFirst);
  },
);
```

:::

## おわりに

StateProvider等ではなくChangeNotifierProviderを採用することがポイントです。
また外部から不要に `oldItem` を更新されないよう `select` などの専用メソッドを用意することも重要です。
最後までお読みいただきありがとうございます。