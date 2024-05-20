---
title: "SwiftLintを歴史あるプロジェクトに導入するための最高のスクリプト"
emoji: "🕰️"
type: "tech"
topics: [Swift, SwiftLint, iOS, Xcode]
published: true
---

## 解決したい問題

歴史のあるプロジェクトにSwiftLintを後から導入すると大量のエラーやワーニングが発生します。
既に導入済みのプロジェクトで後から新しいルールを有効化するような場合にも同様の問題が発生することがあります。

もちろん最善策はルールに従って全てのコードを修正することですが、数百にも及ぶエラー全てに対応する事は現実的ではありません。
修正しないとしても、エラーを無視するために `// swiftlint:disable` というコメントを手作業で全箇所に追加するのも大変な作業です。
ルールの有効化を諦めざるを得ないことも少なくありません。

そこで今回、問題がある全てのコードに対して `// swiftlint:disable:this` を追加するスクリプトを作成しました。

## スクリプト

:::message
JSON解析用コマンド `jq` を利用します。
スクリプト実行前にインストールしてください。
https://formulae.brew.sh/formula/jq
:::

下記のスクリプトを全文コピーしてターミナルで実行すれば動作します。

```sh
swiftlint --quiet --reporter json | \
jq -r '.[] | "\(.file):\(.line):\(.rule_id)"' | \
while IFS=: read -r file line rule_id; do
    target_line=$(sed -n "${line}p" "$file")
    if echo "$target_line" | grep -q "swiftlint:disable:this"; then
        if ! echo "$target_line" | grep -q "swiftlint:disable:this.*${rule_id}"; then
            sed -i '' -e "${line}s/$/ ${rule_id}/" "$file"
        fi
    else
        sed -i '' -e "${line}s/$/ \/\/ swiftlint:disable:this ${rule_id}/" "$file"
    fi
    echo $(sed -n "${line}p" "$file") | sed "s/\/\/ swiftlint:disable:this.*/\x1b[32m&\x1b[0m/"
done
```

:::details CocoaPodsの場合
```sh
Pods/SwiftLint/swiftlint --quiet --reporter json | \
jq -r '.[] | "\(.file):\(.line):\(.rule_id)"' | \
while IFS=: read -r file line rule_id; do
    target_line=$(sed -n "${line}p" "$file")
    if echo "$target_line" | grep -q "swiftlint:disable:this"; then
        if ! echo "$target_line" | grep -q "swiftlint:disable:this.*${rule_id}"; then
            sed -i '' -e "${line}s/$/ ${rule_id}/" "$file"
        fi
    else
        sed -i '' -e "${line}s/$/ \/\/ swiftlint:disable:this ${rule_id}/" "$file"
    fi
    echo $(sed -n "${line}p" "$file") | sed "s/\/\/ swiftlint:disable:this.*/\x1b[32m&\x1b[0m/"
done
```
:::

![](/images/swiftlint-disable-script/1.png)

## 解説

SwiftLintの実行結果から問題があるコードの場所を特定し、その行の末尾にdisableコメントを追加します。
複数のルールで発生するエラーやワーニングについてもまとめて対応可能です。

```sh
# SwiftLintの実行結果をJSON形式で出力
swiftlint --quiet --reporter json | \
# JSONをパースして、問題があるファイル名、行番号、ルールIDを抽出
jq -r '.[] | "\(.file):\(.line):\(.rule_id)"' | \
while IFS=: read -r file line rule_id; do
    target_line=$(sed -n "${line}p" "$file")
    if echo "$target_line" | grep -q "swiftlint:disable:this"; then
        # 既にdisableコメントがある場合は既存のコメントに対してルール追加
        if ! echo "$target_line" | grep -q "swiftlint:disable:this.*${rule_id}"; then
            # 同じルールが重複しないように確認してから追加
            sed -i '' -e "${line}s/$/ ${rule_id}/" "$file"
        fi
    else
        # disableコメントを新規追加
        sed -i '' -e "${line}s/$/ \/\/ swiftlint:disable:this ${rule_id}/" "$file"
    fi
    # 結果の出力
    echo $(sed -n "${line}p" "$file") | sed "s/\/\/ swiftlint:disable:this.*/\x1b[32m&\x1b[0m/"
done
```

追加されたコメントによって `line_length` ルールに抵触する可能性があります。
その場合はスクリプト実行後に個別で対応してください。
または再度スクリプトを実行することでdisableコメントを追加することも可能です。

## 私見

disableコメントで対応してしまうこと、無条件で大量に追加してしまうこと、については賛否あると思います。
私としてはLintは **未来に追加されるコードへの対策** だと捉えているので、既存のコードについては優先度が下がります。

大量のエラーを目にしてルール自体を無効化してしまうよりは

- 既存コードに対してはdisableコメントで対応
- 新規コードに対してはルール有効化

とできる方が良いと考えています。

## おすすめの使い方

とは言っても、実際に全てのエラーをdisableコメントで対応してしまうのは横暴だと思います。

- `--fix --format` で自動フォーマットを利用する
- 正規表現や置換などで一律に処理できるものは対応する
- 件数の少ないものについてはまずは修正を試みる

などで対応した後、それでもエラーが残る場合に今回のスクリプトを利用するのが良いです。

#### 例

たとえば下記のような `.swiftlint.yml` を用意しましょう。
`all` で全てのルールを有効化し、不要なルールのみ無効化する方針をおすすめします。
現状のコードでエラーやワーニングが発生する場合は、コメント付きで無効化しておきます。

```yml:.swiftlint.yml
opt_in_rules:
    - all

disabled_rules:
    - prefer_nimble # XCTestを使う
    - required_deinit # deinitは省略する
    # ...

    # TODO: 以降のルールは一時的に無効化しているため、有効化およびコードの修正をする
    - leading_whitespace
    - private_outlet
    - force_unwrapping
    # ...
```

##### [leading_whitespace](https://realm.github.io/SwiftLint/force_unwrapping.html)

`Supports autocorrection: Yes` なので、有効化して自動フォーマットを利用しましょう。

##### [private_outlet](https://realm.github.io/SwiftLint/private_outlet.html)

`@IBOutlet var` -> `@IBOutlet private var` などの文字列置換で簡単に対応できるので、まとめて対応しましょう。

##### [force_unwrapping](https://realm.github.io/SwiftLint/force_unwrapping.html)

有効化して大量にワーニングが発生する場合、出来ることは少ないです。
今回のスクリプトを利用してdisableコメントを追加しましょう。

## あとがき

私はシェルや正規表現に苦手意識を持っているので、今回のように少し複雑なスクリプトはこれまで避けてきました。
最近はこういった関心が低い作業についてはさらっとAIに任せることができるので、自身は本質的なことだけに集中できて良いですね。

ちなみに、私はSwiftLintの自動フォーマットはpre-commitで行うのが好きです。