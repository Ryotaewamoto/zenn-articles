---
title: "The Non-Designer's Color Schemes" # 記事のタイトル
emoji: "🍎" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Dart", "Android", "Web", "個人開発"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに
ご覧いただきありがとうございます。[gan](https://zenn.dev/ryota_iwamoto)です。

何かアプリを作る際の色の定義って大変ですよね。メインの色以外にも、背景色だったりちょっとアクセントを加えたい色だったり、、、。最初にいくつか考えていても開発してる最中に別の色が必要になったりして、アプリの色に関して開発中も結構考えちゃってませんか？（それはまさに今の自分なのですが）

そこで、僕はこう思いました。

**デザイナーじゃないけど、いい感じの色の定義を最初にしちゃいたい！！**

ということで、この記事では個人開発やデザイナーが苦手な人向けにうってつけの Web サイトがあるので紹介します！

:::message
自分自身が Flutter エンジニアなのでそちらの目線になりますが、Android や Web でも使える技術ですので最後まで見ていただけると幸いです。また、あくまでエンジニア目線での記事ですので、デザイナーさんから見たら、「それは違うよ！」というところがあるかもしれません。その場合はコメントしていただけると、とても嬉しいです！
:::

# 今回紹介するもの

先にサイトのリンクを貼っておきます。

**いいねを押して、以下のリンクに飛んでもらえればこの記事の役目を終了です。**

**お疲れ様でした！**

となって良いのですが、せっかくなのでいくつか言葉の意味や使い方を解説します。めっちゃ簡単なんで安心してください！！

https://m3.material.io/theme-builder#/custom

## 定義 (Roles in a scheme)

以下では色を決める際に出てくる言葉の定義をします。詳細は下のリンクからも見れますので詳しく知りたい方は覗いてみてください。
せっかくなので、この記事では具体例を多めでいこうと思います。

https://m3.material.io/styles/color/the-color-system/color-roles

### Primary

Primary は、主にアプリのメインの部分の色になります。FAB（Floating Action Button）やメインのボタン、目立たせたいテキストの色などがあたります。ロゴの色に合わせるのも良いです。また、On Primary はその上の文字やアイコンの色になります。Primary Container は目立たせたいカードなどの中身を埋めるような色になります。On Primary Container は同様に、その上の文字やアイコンの色になります。

![](https://storage.googleapis.com/zenn-user-upload/3b5d248c7383-20220926.png)

![](https://storage.googleapis.com/zenn-user-upload/bfe39d903907-20220926.png)

#### 実例: Twitter

![](https://storage.googleapis.com/zenn-user-upload/73a7d15e614c-20220926.png)

### Secondary

Secondary は主に Primary の次にアクセントを加えたい色になります。ただ主観として、いろんなアプリを見てみましたが、Secondary がないアプリも多く存在しています。なので Secondary が絶対必要！というわけではないようです。
また、On Secondary や Secondary Container、On Secondary Container の説明は Primary と同じなので省略します。

![](https://storage.googleapis.com/zenn-user-upload/69f7dbf3ec8b-20220926.png)

![](https://storage.googleapis.com/zenn-user-upload/fe22d91f31a2-20220926.png)

#### 実例: Starbucks

![](https://storage.googleapis.com/zenn-user-upload/e74be229a760-20220926.png)

### Tertiary

Tertiary は日本語で、「3次の」や「第3位の」という意味があります。その意味の通り、3番目にアクセントをつけたい部分に使う色が Tertiary になります。この色は Primary や Secondary とは対照的な色になります。主な使用場面としては、テキストフィールドのアクセントカラーや優先度のあまり高くないチェックボックスが挙げられます。

![](https://storage.googleapis.com/zenn-user-upload/5ecf6f977833-20220926.png)

![](https://storage.googleapis.com/zenn-user-upload/c4ee90855283-20220926.png)

#### 実例: Slack

![](https://storage.googleapis.com/zenn-user-upload/815b8ae32a4b-20220926.png)

### Neutral

Neutral は背景色や文字を強調したい時に使う色になります。

:::message alert
この記事を書いている中で気づいたのは、**TextColor** のような色は存在しない、ということです。テキストの色を一つに決めるのではなく背景の色に対して On ~ のようにして文字の色を決定させるのが Material Design のようです。
:::

# サイトの使い方

上のリンクを押してもらうと、以下のようなページが表示されると思います。

![](https://storage.googleapis.com/zenn-user-upload/3b34813736de-20220925.png)

ここで左側の部分で好きな色を決めることができます。その色が真ん中のデバイスに反映されます。右端では決めた色の16進数の形で見ることができます。また右上の「Export」を押すことで欲しい形式のファイルを生成してくれます。

![](https://storage.googleapis.com/zenn-user-upload/6875206f783a-20220926.png)

今回は **Flutter（Dart）** を選択してみます。（ CSS(Web) や XML(Android) の形式でもできます。）すると zip ファイルがダウンロードされるのでそれを開きます。中身は設定した色の定義ファイル（color_schemes.g.dart）とサンプルのUI（main.g.dart）になります。

::::details color_schemes.g.dart
```dart
import 'package:flutter/material.dart';

const lightColorScheme = ColorScheme(
  brightness: Brightness.light,
  surfaceTint: Color(0xFF6750A4),
  onErrorContainer: Color(0xFF410E0B),
  onError: Color(0xFFFFFFFF),
  errorContainer: Color(0xFFF9DEDC),
  onTertiaryContainer: Color(0xFF31111D),
  onTertiary: Color(0xFFFFFFFF),
  tertiaryContainer: Color(0xFFFFD8E4),
  tertiary: Color(0xFF7D5260),
  shadow: Color(0xFF000000),
  error: Color(0xFFB3261E),
  outline: Color(0xFF79747E),
  onBackground: Color(0xFF1C1B1F),
  background: Color(0xFFFFFBFE),
  onInverseSurface: Color(0xFFF4EFF4),
  inverseSurface: Color(0xFF313033),
  onSurfaceVariant: Color(0xFF49454F),
  onSurface: Color(0xFF1C1B1F),
  surfaceVariant: Color(0xFFE7E0EC),
  surface: Color(0xFFFFFBFE),
  onSecondaryContainer: Color(0xFF1D192B),
  onSecondary: Color(0xFFFFFFFF),
  secondaryContainer: Color(0xFFE8DEF8),
  secondary: Color(0xFF625B71),
  inversePrimary: Color(0xFFD0BCFF),
  onPrimaryContainer: Color(0xFF21005D),
  onPrimary: Color(0xFFFFFFFF),
  primaryContainer: Color(0xFFEADDFF),
  primary: Color(0xFF6750A4),
);

const darkColorScheme = ColorScheme(
  brightness: Brightness.dark,
  surfaceTint: Color(0xFFD0BCFF),
  onErrorContainer: Color(0xFFF2B8B5),
  onError: Color(0xFF601410),
  errorContainer: Color(0xFF8C1D18),
  onTertiaryContainer: Color(0xFFFFD8E4),
  onTertiary: Color(0xFF492532),
  tertiaryContainer: Color(0xFF633B48),
  tertiary: Color(0xFFEFB8C8),
  shadow: Color(0xFF000000),
  error: Color(0xFFF2B8B5),
  outline: Color(0xFF938F99),
  onBackground: Color(0xFFE6E1E5),
  background: Color(0xFF1C1B1F),
  onInverseSurface: Color(0xFF313033),
  inverseSurface: Color(0xFFE6E1E5),
  onSurfaceVariant: Color(0xFFCAC4D0),
  onSurface: Color(0xFFE6E1E5),
  surfaceVariant: Color(0xFF49454F),
  surface: Color(0xFF1C1B1F),
  onSecondaryContainer: Color(0xFFE8DEF8),
  onSecondary: Color(0xFF332D41),
  secondaryContainer: Color(0xFF4A4458),
  secondary: Color(0xFFCCC2DC),
  inversePrimary: Color(0xFF6750A4),
  onPrimaryContainer: Color(0xFFEADDFF),
  onPrimary: Color(0xFF381E72),
  primaryContainer: Color(0xFF4F378B),
  primary: Color(0xFFD0BCFF),
);

```
::::

::::details main.g.dart
```dart

import 'package:flutter/material.dart';
import 'color_schemes.g.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(useMaterial3: true, colorScheme: lightColorScheme),
      darkTheme: ThemeData(useMaterial3: true, colorScheme: darkColorScheme),
      home: const Home(),
    );
  }
}

class Home extends StatelessWidget {
  const Home({Key? key}) : super(key: key);

  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          elevation: 2,
          title: Text("Material Theme Builder"),
        ),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const Text(
                'Update with your UI',
              ),
            ],
          ),
        ),
        floatingActionButton:
            FloatingActionButton(onPressed: () => {}, tooltip: 'Increment'));
  }
}

```
::::

この Color_schemes.g.dart の使い方は簡単で使いたいところで以下のように呼んであげるとダークモードにも対応した色の設定になります。(AppBar の背景色を例にしています)

```dart
return MaterialApp(
    title: 'Flutter Demo',
    theme: ThemeData(useMaterial3: true, colorScheme: lightColorScheme),
    darkTheme: ThemeData(useMaterial3: true, colorScheme: darkColorScheme),
    ...
```

```dart
@override
Widget build(BuildContext context) {
return Scaffold(
    appBar: AppBar(
    backgroundColor: Theme.of(context).colorScheme.primary,
    title: const Text('Screen A'),
    ),
    ...
```

これによって、アプリ開発での色に関する悩みが少しでも軽減されたら嬉しいです！

# 最後に

本記事は以下のコンテンツを参考にさせていただきました。ありがとうございます！

https://codelabs.developers.google.com/codelabs/flutter-boring-to-beautiful?hl=ja#0

https://m3.material.io/styles/color/the-color-system/color-roles

https://mobbin.com/browse/ios/apps
