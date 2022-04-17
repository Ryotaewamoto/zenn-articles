---
title: "【Flutter】脱print文" # 記事のタイトル
emoji: "👶" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# はじめに
ご覧いただきありがとうございます。[gan](https://zenn.dev/ryota_iwamoto)です。

今回は皆さん一度は使ったことのあるprint文についてです。
Flutterでアプリ開発をしている際に
- この変数にちゃんと値が入ってるかな？
- データベースからjson形式のデータを取れてるかな？
- ElevatedButtonを押したときにちゃんと反応してるか確かめたい
- ここは分かりづらいからログを出しておきたいな

などprint文を使いたい、もしくは使っている場面を多くあると思います。ですが、この記事ではprint文を使わずに済むような方法を紹介していきます。
この記事を参考にして読んでくださった方がprint文から解放されたアプリ開発をできることを楽しみにしています。

# print文の問題点

まずはprint文の何が悪いのかについて書いておきます。2つあります。

1. コードの肥大化
2. コンソールの煩雑化

一つ目は単純でprint文を多用することでコードは肥大化します。例えばですが、

```dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  MyHomePage({Key key, this.title}) : super(key: key);

  final String title;

  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
      print('カウンターの値を出力: $_counter'); // ココ
    });
  }

  void _decrementCounter() {
    setState(() {
      _counter--;
      print('カウンターの値を出力: $_counter'); // ココ
    });
  }

  @override
  Widget build(BuildContext context) {
    print('MyHomePageをビルド'); // ココ
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(
              'You have pushed the button this many times:',
            ),
            Text(
              '$_counter',
              style: Theme.of(context).textTheme.display1,
            ),
            ElevatedButton(
              child: Text('Decrement'),
              onPressed: _decrementCounter,
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: Icon(Icons.add),
      ), // This trailing comma makes auto-formatting nicer for build methods.
    );
  }
}
```

サンプルプログラムに少し毛を生やしたものです。この程度のコード量であればあまりに気にならないですが、コードが増えていくうちにprint文が積み重なっていきます。チリツモです。またその際にコンソールの煩雑化がおきます。イメージとしては、

```
flutter: ○○○○○○○○○○
flutter: ○○○○○○○○○○○○○○
flutter: ○○○○○○○○○○○○
flutter: ○○○○○○○○○○
flutter: ○○○○○○○○○○○○○○○○○○
flutter: ○○○○○○○○○○○
flutter: ○○○○○○○○○○○○○
flutter: ○○○○○○○○○○
flutter: ○○○○○○○○○○○○○○
flutter: ○○○○○○○○○○○○○○○○○
flutter: ○○○○○○○○○○
```

こんな感じ？です。どこで何が起きているのか何のためのログなのか分かりづらくなっていきます。print文の文章を工夫すればいいと思う方もいると思います。ですが、文章を変えるとテキストの長さは延び、読む側のコストも高くなるので開発効率はあまり良くありません。また非常に個人的ではありますが、エラーコードが出たときに邪魔です。シンプルに見づらい。



# 今回の内容

# まとめ