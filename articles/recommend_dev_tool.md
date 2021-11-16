---
title: "【初心者向け】Dev Tools を使いこなそう" # 記事のタイトル
emoji: "🧸" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Dart"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# はじめに
ご覧いただきありがとうございます。ganです。

今回はFlutter を使うなら欠かせない『Dev Tools』について書いていきます。自分は最近使い始めたのですが、もうこれがないと生きていけません。（それくらい便利です！）

なるべく詳しく書くのでぜひ使ってみてください！！


# Dev Tools とは？

はじめにDev Tools とは何かを説明します。

Dev Tools というのはFlutter やDart を使って開発をする人向けのもので、アプリのレイアウト検証(layout inspection)やパフォーマンス(performance), メモリ、またデバッグなどを効率よくとこなうためのWebアプリケーションツールです。

かなり便利な機能がいくつもありますが、今回はレイアウト検証(layout inspection)について重点的に解説します。


参考動画
（細かい操作方法も教えてくれているのでかなりおすすめです）

https://www.youtube.com/watch?v=nq43mP7hjAE

:::message alert
この記事ではAndroidStudio を使って説明していきます。
:::

:::message
VS Code の方は以下の記事が参考になると思います。
:::

https://docs.flutter.dev/development/tools/devtools/vscode


# 使い方

## 起動方法

これはとても簡単でツールウィンドウのDebug を選択します。
開いたらConsole と書いてある行の右端にあるDart のアイコンをクリックします。
するとブラウザでDev Tools が起動します。

![](https://storage.googleapis.com/zenn-user-upload/915aaf94138e-20211117.png)

クリックすると、

![](https://storage.googleapis.com/zenn-user-upload/05d065fdc795-20211117.png)

この画面が現れます。
ここまできたら準備完了です。

## 機能.1 Flutter Inspector

これから実際にDev Tools を使っていこうと思います。

サンプルとして以下のようなアプリを使用します。

![](https://storage.googleapis.com/zenn-user-upload/eaf0badfa9d1-20211117.png)

コード
```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatelessWidget {
  const MyHomePage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Dev Toolsを使ってみよう！！'),
      ),
      body: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          Row(
            children: [
              ElevatedButton(onPressed: () {}, child: const Text('buttonA')),
              ElevatedButton(onPressed: () {}, child: const Text('buttonB')),
              ElevatedButton(onPressed: () {}, child: const Text('buttonC')),
            ],
          ),
          Image.network(
            'https://www.w2solution.co.jp/tech/wp-content/uploads/2020/05/flutter-top.png',
            height: 150,
            fit: BoxFit.fill,
          ),
          const SizedBox(
            height: 16,
          ),
          const Text(
            'Dev Toolsを使ってみよう！',
          ),
          const Text(
            'My application is great! Everyone often uses it in my classroom.',
          ),
        ],
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {},
        tooltip: 'Increment',
        child: const Icon(Icons.add),
      ), // This trailing comma makes auto-formatting nicer for build methods.
    );
  }
}

```

まずはDev Tools の画面左にあるWidgetツリーのElevatedButtonをクリックします。
すると画面右がこのように変わります。

![](https://storage.googleapis.com/zenn-user-upload/3e45eee88e55-20211117.png)





# まとめ