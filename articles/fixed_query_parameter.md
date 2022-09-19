---
title: "検索クエリパラメータのトリミング？" # 記事のタイトル
emoji: "🔍" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Dart", "API"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

![](https://storage.googleapis.com/zenn-user-upload/05948c1ad658-20220919.jpg)

# はじめに
ご覧いただきありがとうございます。[gan](https://zenn.dev/ryota_iwamoto)です。

検索に関する API を利用する際に、いくつか困ったことがあったのでもろもろ共有しようと思います。

ベストプラクティスがあれば教えていただけると嬉しいです！

また記事のタイトルについて、トリミング以外の処理も行っているので末尾に「？」をつけています。適切な名前がわかる方がいましたら教えてください！

# 解決したい問題

ユーザが何かを検索する際に入力しそうな文字列を、クエリパラメータに入れられる適切な文字列に変更したい。

ここでユーザが何かを検索する際に入力しそうな文字列とは、

- 検索したい文字列の前にスペースがある
- 複数の情報で検索しようとして、それぞれの文字の間に（全角また半角）スペースがある（例えば、「山 海」みたいな感じ）

その上で、クエリパラメータに入れられる適切な文字列とは、

例えば「 山　sea 雲」(半角スペース+文字+全角スペース+文字+半角スペース+文字) であれば、

山+sea+雲

# 実装

基本的にフロント側で行う処理であると思うので、言語は Dart を使って書きます。（TypeScriptとかでもだいたい同じな気がする）

必要であればコピペして使ってください。

```dart
/// ユーザが入力した文字列をクエリパラメータに変換する関数
/// 引数には、ユーザによって入力された文字列が入る
String _validateQueryParameter(String text) {
    // テキストの始めと終わりにあるスペースを削除する。
    final trimmedText = text.trim();

    // スペースが含まれない場合はそのまま返す。
    if (!trimmedText.contains(' ') && !trimmedText.contains('　')) {
        return trimmedText;
    }

    // 半角スペースが含まれる場合
    if (!trimmedText.contains('　')) {
        final separatedTexts = trimmedText.split(' ');
        String q = separatedTexts[0];
        for (final text in separatedTexts) {
            if (text == q) {
                // 初めの文字に対して処理を行わない。
            } else {
                q = '$q\+$text';
            }
        }
        return q;
    }

    // 全角スペースも含まれる場合
    final weekSeparatedTexts = trimmedText.split(' ');
    List<String> strengthSeparatedTexts = [];
    for (final text in weekSeparatedTexts) {
        if (text.contains('　')) {
            final elementSeparatedTexts = text.split('　');
            for (final text in elementSeparatedTexts) {
                strengthSeparatedTexts.add(text);
            }
        } else {
            strengthSeparatedTexts.add(text);
        }
    }
    String q = strengthSeparatedTexts[0];
    for (final text in strengthSeparatedTexts) {
        if (text == q) {
            // 初めの文字に対して処理を行わない。
        } else {
            q = '$q\+$text';
        }
    }
    return q;
}
```

# 懸念点

- いくつかパターンを考えたがアンチパターンが存在する可能性
- すべて immutable(不変)にするとコード量が増えるので、for文周りは mutable(可変)

# まとめ
Google 検索はすばらしい
