---
title: "【Flutter】最後にちょっとだけ実装してみませんか？" # 記事のタイトル
emoji: "🛩" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Dart"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに
ご覧いただきありがとうございます。ganです。
今回は、「アプリはあと少しで完成しそう！」って時に実装したいものを3つ紹介します。
個人アプリを作っているなどぜひ参考にしてみてください。
また自分はこういうのも使ってるよ！っていうものがあればコメントしていただけると嬉しいです！

# 1. Haptic Feedback

1つ目はHaptic Feedbackです。これは何かというと、スマホを振動させる機能です！実装はとても簡単ですが意外とあなどれないなと思ったので紹介します。なんならこれを紹介するためにこの記事を書いたと言っても過言ではありません笑。アプリに合わせて要所に使ってみるといいと思います。

:::message
simulatorやemulatorでは振動しないので実機でビルドして確認してみてください。
:::

## 使い方

まず、以下をファイルに追加してインポートします。
```dart
import 'package:flutter/services.dart';
```
そしたらあとは振動させたいところに``HapticFeedback.mediumImpact();``を追加します。
```dart
  void _mediumHaptic() {
    setState(() {
      HapticFeedback.mediumImpact(); // ココ！
    });
  }
```

androidの場合は``android/app/src/main/AndroidManifest.xml``に以下の文を追加してください。
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="ダミー">
    <!-- haptic feedbackの権限要求(ココ！) -->
    <uses-permission android:name="android.permission.VIBRATE" /> 
```

補足：``mediumImpact``以外のメソッドとして、``heavyImpact``, ``lightImpact``, ``vibrate``, ``selectionClick``(連続的な振動)があります。

# 2. Confetti(紙吹雪)

2つ目はConfetti(紙吹雪)です。これはシンプルに紙吹雪を追加するということです。ユーザが何か嬉しくなるようなアクションを起こした際に紙吹雪があるとかなりいいユーザ体験を得られると思います。こちらはパッケージを使うので簡単に実装できます！

![Confetti](https://user-images.githubusercontent.com/75112184/161222363-ff4c45fa-37be-4055-827b-547547125755.gif)

:::message
以下のコードでのgifになります。
:::

## 使い方

まずはパッケージを追加しましょう！

https://pub.dev/packages/confetti


次に紙吹雪を使いたい画面にコードを書いていきます。はじめにcontrollerを追加して紙吹雪を出したいところで``play``メソッドを呼びます。controllerの引数には紙吹雪を出す時間を入れてください。
そしたら紙吹雪を出したいところで``ConfettiWidget``を呼んであげます。``Align``ウィジェットで位置を指定してあげるといいと思います。

```dart
class MyPage extends StatefulWidget {
  const MyPage({Key? key}) : super(key: key);

  @override
  State<MyPage> createState() => _MyPageState();
}

class _MyPageState extends State<MyPage> {
  final _controller = ConfettiController(duration: const Duration(seconds: 5)); // ココ！

  void _confettiEvent() {
    setState(() {
      _controller.play(); // ココ！
    });
  }

  void _functionA() {
    Future.
  }

  @override
  void dispose() {
    _controller.dispose(); // controllerを破棄する
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('タイトル'),
      ),
      body: Center(
        child: Stack(
          children: <Widget>[
            Center(
              child: Column(
                children: <Widget>[
                  Text(
                    _centerText,
                    style: Theme.of(context).textTheme.headline4,
                  ),
                  ElevatedButton(
                    onPressed: _confettiEvent, // ココで紙吹雪を呼び出し
                    child: const Text('Push here!'),
                  ),
                ],
              ),
            ),
            Align(
              alignment: Alignment.bottomCenter,
              child: ConfettiWidget(
                confettiController: _controller,
                blastDirection: pi / 2, // 紙吹雪を出す方向(この場合画面上に向けて発射)
                emissionFrequency: 0.1, // 発射頻度(数が小さいほど紙と紙の間隔が狭くなる)
                minBlastForce: 5, // 紙吹雪の出る瞬間の5フレーム分の速度の最小
                maxBlastForce: 20, // 紙吹雪の出る瞬間の5フレーム分の速度の最大(数が大きほど紙吹雪は遠くに飛んでいきます。)
                numberOfParticles: 10, // 1秒あたりの紙の枚数
                gravity: 0.1, // 紙の落ちる速さ(0~1で0だとちょーゆっくり)
                colors: const <Color>[ // 紙吹雪の色指定
                  Colors.red,
                  Colors.blue,
                  Constants.green,
                ],
              ),
            )
          ],
        ),
      ),
    );
  }
}
```

他のConfettiWidgetの他の引数が気になる方、以下の記事で動画付きで解説しているのでぜひ見てみてください。

https://blog.dalt.me/2478

# 3. Percent Indicator

デフォルトのインジケータではなく進み具合を表示させるインジケータを使ってみよう、ということです！重たい処理をする時にローディングを出してあげるのは非常にいいことです。ただせっかくなら進み具合も表示して欲しい！とユーザは潜在的に思ってるかも？ということで進み具合に合わせたインジケータを使ってみてはどうでしょうか？こちらもパッケージがあるのですぐにできて便利です。

円以外にも直線など色々あるので色々試してみてください！

![Percent Indicator](https://storage.googleapis.com/zenn-user-upload/fc406e5d3930-20220331.gif)

## 使い方

まずはパッケージを追加しましょう！

https://pub.dev/packages/percent_indicator

状態管理？はとりあえずStatefulWidgetを使って書きます。使用している状態管理方法に合わせて使用してください。

StatefulWidgetでももっと綺麗に書く方法はあると思いますので、実装する場合はうまく調整してみてください。

```dart
class MyHomePage extends StatefulWidget {
  const MyHomePage({Key? key}) : super(key: key);

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  bool isLoading = false;

  double _loadingCounter = 0; // この値を変化させていく

  @override
  void initState() {
    super.initState();
  }

  @override
  void dispose() {
    super.dispose();
  }

  // 時間のかかる処理
  void bigFunction() async {
    // ここでローディング開始
    setState(() {
      _loadingCounter = 0;
      isLoading = true;
    });
    setState(() {
      _loadingCounter = 0.1; 
    });
    await functionA();
    setState(() {
      _loadingCounter = 0.3; // 処理が終わるごとに_loadingCounterの値を大きくしていく
    });
    await functionB();
    setState(() {
      _loadingCounter = 0.7;
    });
    await functionC();
    setState(() {
      _loadingCounter = 1;
    });
    await functionLast();
    setState(() {
      isLoading = false;
    });
  }

  // 適当な処理A
  Future<void> functionA() async {
    await Future.delayed(const Duration(seconds: 1));
  }

  // 適当な処理B
  Future<void> functionB() async {
    await Future.delayed(const Duration(seconds: 1));
  }

  // 適当な処理C
  Future<void> functionC() async {
    await Future.delayed(const Duration(seconds: 1));
  }

  // 最後インジケータが一周するように微調整
  Future<void> functionLast() async {
    await Future.delayed(const Duration(milliseconds: 500));
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Flutter Demo Home Page'),
      ),
      body: Stack(
        children: [
          DecoratedBox(
            decoration: const BoxDecoration(
              gradient: LinearGradient(
                begin: Alignment.topLeft,
                end: Alignment.bottomRight,
                colors: <Color>[
                  Colors.white,
                  Colors.greenAccent,
                ],
              ),
            ),
            child: Center(
              child: !isLoading ? Stack(
                // mainAxisAlignment: MainAxisAlignment.center,
                children: <Widget>[
                  if (!isLoading)
                  Center(
                    child: Column(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        ElevatedButton(
                          onPressed: () async {
                            bigFunction(); // ここで関数を呼び出してます。
                          },
                          child: const Text('Hey! Push!!'),
                        ),
                      ],
                    ),
                  ),
                ],
              ) : Center(
              child: CircularPercentIndicator( // ここの引数でデザインを変更することもできます。
                progressColor: Colors.green,
                animateFromLastPercent: true,
                animation: true,
                radius: 50,
                percent: _loadingCounter,
              ),
            )
            ),
          ),            
        ],
      ),
    );
  }
}
```

# まとめ
いかがだったでしょうか？
一つでも「これ使えそう！」と思っていただければ嬉しいです！
