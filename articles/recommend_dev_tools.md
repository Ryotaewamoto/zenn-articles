---
title: "【Flutter】【初心者向け】Dev Tools を使ってみよう！" # 記事のタイトル
emoji: "🧸" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Dart"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに
ご覧いただきありがとうございます。ganです。

今回はFlutter を使うなら欠かせない『Dev Tools』について書いていきます。自分は最近使い始めたのですが、もうこれがないと生きていけません。（それくらい便利です！）

なるべく画像多めで詳しく書いたので、使ってなかったよって人はぜひ使ってみてください！！


# Dev Tools とは？

はじめにDev Tools とは何かを説明します。

Dev Tools というのはFlutter やDart を使って開発をする人向けのもので、アプリのレイアウト検証(layout inspection)やパフォーマンス(performance), メモリ、またデバッグなどを効率よくとこなうためのWebアプリケーションツールです。

かなり便利な機能がいくつもありますが、今回はレイアウト検証(layout inspection)について重点的に解説します。

:::message alert
この記事ではAndroidStudio を使って説明していきます。
:::

:::message
VS Code の方は以下の記事が参考になると思います。
:::

https://docs.flutter.dev/development/tools/devtools/vscode


# 使い方

## サンプルコード

サンプルとして以下のようなアプリを使用します。

![](https://storage.googleapis.com/zenn-user-upload/c3f2eef4a827-20211117.png)

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
      ), 
    );
  }
}

```

## 起動方法

これはとても簡単でツールウィンドウのDebug を選択します。
開いたらConsole と書いてある行の右端にあるDart のアイコンをクリックします。
するとブラウザでDev Tools が起動します。

![](https://storage.googleapis.com/zenn-user-upload/915aaf94138e-20211117.png)

クリックすると、

![](https://storage.googleapis.com/zenn-user-upload/05d065fdc795-20211117.png)

この画面が現れます。
ここまできたら準備完了です。

## layout inspection

これから実際にDev Tools を使っていこうと思います。

### 基本的な使い方

まずはDev Tools の画面左にあるWidgetツリーのElevatedButtonをクリックします。
すると画面右がこのように変わります。

![](https://storage.googleapis.com/zenn-user-upload/3e45eee88e55-20211117.png)

ぜひ自分のアプリを開発している場合はそちらでもやってほしいです。

ここで驚きなのは、各Widgetのwidth(横幅)とheight(高さ)が明らかなことです。
これによってpixel単位の開発が可能になります。
またMain AxisやCross Axisの値も表示されていてありがたいですね。

ちなみにですが、選択したWidgetは画面下のConsoleにも表示されます。

### Select Widget Mode（オススメ！）

次に、画面左上にある『Select Widget Mode』を押してみます。

![](https://storage.googleapis.com/zenn-user-upload/2d66f21765c8-20211117.png)

すると画像のようにボタンが青くなります。この状態でシミュレータの画像部分を押すと、以下のような画面になります。

![](https://storage.googleapis.com/zenn-user-upload/2e2b0e5a9bd0-20211117.png)

![](https://storage.googleapis.com/zenn-user-upload/3d9fe098357b-20211117.png)

このように『Select Widget Mode』を使うことで実際の端末ではどのようにWidgetが配置されているのかを確認することができます。

実際にサンプルアプリの場合、画像の高さ(height)がちゃんと150.0になっているのが確認できます。

もし他のWidgetを確認したい場合はシミュレータの左下にある検索ボタンを押すと、再度Widgetを選択できる状態になります。

![](https://storage.googleapis.com/zenn-user-upload/dc46d65f299f-20211117.png)


さらにこの機能でオススメなのはRowやColumnを選択した時です！

選択した場合、以下のような画面になります。ここでDev Toolsの画面右上のMain Axis と書かれたとこの右にあるstartの横のボタンを押します。

![](https://storage.googleapis.com/zenn-user-upload/eefb9f041dbc-20211117.png)

![](https://storage.googleapis.com/zenn-user-upload/5b0eb1640915-20211117.png)

ここで例えば、spaceBetweenを選択すると、

![](https://storage.googleapis.com/zenn-user-upload/53fe2c159a48-20211117.png)

このようにシミュレータの方にも反映されます!
なのでMainAxisAlignmentの値を変更して、hot reloadしなくても変更した時にどのようにWidgetが配置されるかを確認できます。

Columnを選択してcenterからspaceEvenlyに変更した場合、以下のようになります。

![](https://storage.googleapis.com/zenn-user-upload/509f8385e248-20211117.png)

:::message alert
元々のファイルのソースコードは変更されないので、見た目が確認できたら変更するようにしましょう。
:::

最後に、Details TreeをクリックするとWidgetの引数の値が見れるので地味にこれもオススメです。

![](https://storage.googleapis.com/zenn-user-upload/31c4e2d420bd-20211117.png)

# 終わりに

Dev Toolsには他にもパフォーマンスを調べたり、メモリの使用量を動作ごとに確認したりすることがきます。自分はまだlayout inspectionしか使えていませんが、これからはアプリの速度やパフォーマンスにも気を使っていきたいと思います。

より良いFlutterライフを過ごしていきましょう！！

参考動画
（細かい操作方法も教えてくれているのでかなりおすすめです）

https://www.youtube.com/watch?v=nq43mP7hjAE