---
title: "Flutter Web でブラウザのバックボタンを押された時の処理" # 記事のタイトル
emoji: "😥" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Web", "Flutter"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに
ご覧いただきありがとうございます。ganです。

今回はFlutter web でブラウザバックのボタンが押された時に画面遷移したいときの解決策を書きます。


![](https://storage.googleapis.com/zenn-user-upload/595c93cdd33d653532a8f294.png)

# 今回の内容

ブラウザバックボタンが押された時に処理を実行したい。

# 解決法

- build関数のreturnないのWidgetをWillPopWillPopScopeというWidgetで囲みます。

下に具体的な使用方法を書いておきます。

```dart:next_page.dart
class NextPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return WillPopScope(
      onWillPop: ()  => Navigator.push(context, MaterialPageRoute(builder: (context)  => FirstPage())),
      child: Scaffold(
        appBar: AppBar(),
        body: Container(),
      )
    );
  }
}
```

```dart:first_page.dart
class FirstPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(),
      body: Container(),
    );
  }
}
```

この場合、NextPageでブラウザバックボタンが押されるとFirstPageに画面遷移します。

WillPopScope ウィジェット内のonWillpop を変更することで、バックブラウザボタンが押された時の処理を変えることができます。

# まとめ

今回はWillPopScopeというWidgetを使って解決しました。

文章量は少ないですが、これからも少しずつ記事を書いていこうと思います。

# 参考

https://stackoverflow.com/questions/61322093/how-to-configure-go-back-button-in-browser-for-flutter-web-app