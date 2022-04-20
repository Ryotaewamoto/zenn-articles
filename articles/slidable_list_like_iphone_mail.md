---
title: "【Flutter】リストの要素を横にスライドさせたい(iPhoneのメール的な)[2022/04/20時点]" # 記事のタイトル
emoji: "👆" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに
ご覧いただきありがとうございます。[gan](https://zenn.dev/ryota_iwamoto)です。

今回はListViewの要素を横にスライドできるようにしていきたいと思います！
具体的な例としてはiPhoneに元から入っているメールアプリです。

- iPhoneのメールアプリ

![](https://storage.googleapis.com/zenn-user-upload/6390d4f32770-20220420.png =200x)
※自分アカウントのメールなので内容はあまり気にしないでください。

これと似たようなUIをした自作のアプリ*1を使って解説していこうと思います。

- iPhoneのメールアプリをトレースしたもの

![](https://storage.googleapis.com/zenn-user-upload/a559cfce05e1-20220420.png =200x)

今回、**flutter_slidable**というパッケージを使用しました。

https://pub.dev/packages/flutter_slidable

Flutter Favoriteと**公式のお墨付き**です！

ここでこの記事を書いた経緯について、**flutter_slidableパッケージのバージョンが上がったこと**によって、Googleの検索結果の上位の

https://qiita.com/ryota47/items/6ed90ce011ea6aaa8698

https://masamarun.com/flutter-slidable/

の記事がバージョンが上がる前のもので書き方がかなり変わっていたのが理由です。

またこの記事をきっかけにFlutterってこんなこともできるんだ！となってもらえたら嬉しいです！

# flutter_slidableの使い方

まずは以下のgifでどういう動きになるかを確認してみてください。

![](https://storage.googleapis.com/zenn-user-upload/aedb77b7539d-20220420.gif =200x)

基本的な使い方は以下の通りです。

準備: 以下よりパッケージをインストール

https://pub.dev/packages/flutter_slidable/install

(1) listViewの戻り値のWidget(この場合はColumn)をSlidableWidgetでラップする

(2) 左からスライドしたい場合はstartActionPaneの引数を、右からスライドしたい場合はendActionPaneの引数を追加します。(どちらも使い方は同じなので以下startActionPaneのみ説明します。)

(3) ActionPaneのchildrenにはスライドした時に出てきてほしいWidgetを追加します。このときListの要素はすべてSlidableAction型です。以下を参考に自分でカスタマイズしてみてください。

主要なSlidableActionのプロパティ
```
onPressed: タップされた時に起こる処理,
backgroundColor: 背景色,
foregroundColor: アイコンの色,
icon: アイコン,
label: アイコンの下のテキスト
```

(4) startActionPaneのextentRatio引数にどのくらいの幅で表示したいかをdouble型で指定します(0~1)。またSlidableActionのflexという引数を使って相対的に表示することもできます。

(5) startActionPaneのmotion引数に``BehindMotion()``, ``DrawerMotion``, ``ScrollMotion``, ``StretchMotion``

どのような動きかは以下のページのREADMEに記載されているので確認してみてください。

https://pub.dev/packages/flutter_slidable

(6) 最後にスライドしきった時の処理についてですこの時はSlidableのkeyを指定してstartActionPaneのdismissibleにDismissiblePaneを追加します。

:::message alert
この時にonDismissedの関数内でリストの要素も削除した方が良さそうです。おそらく、
``A dismissed Dismissible widget is still part of the tree``と言うエラーが発生し「消したはずのwidgetがまだツリーにあるよ」と怒られます。
:::




```dart
              child: ListView.builder(
                itemCount: messageData.length,
                itemBuilder: (BuildContext context, int index) {
                  final message = messageData[index];

                  return Slidable( // (1)
                    // enabled: false, // falseにすると文字通りスライドしなくなります
                    // closeOnScroll: false, // *2
                    // dragStartBehavior: DragStartBehavior.start,
                    key: UniqueKey(),
                    startActionPane: ActionPane( // (2)
                      extentRatio: 0.2,
                      motion: const ScrollMotion(), // (5)
                      children: [
                        message['isChecked']
                            ? SlidableAction(
                                onPressed: (_) {},
                                backgroundColor: AppColor.mainColor,
                                foregroundColor: AppColor.backgroundColor,
                                icon: AppIcon.markEmailUnreadOutlined,
                                label: 'Unread',
                              )
                            : SlidableAction(
                                onPressed: (_) {},
                                backgroundColor: AppColor.mainColor,
                                foregroundColor: AppColor.backgroundColor,
                                icon: AppIcon.draftsOutlined,
                                label: 'Read',
                              )
                      ],
                    ),
                    endActionPane: ActionPane( // (2)
                      extentRatio: 0.5,
                      motion: const StretchMotion(), // (5)
                      dismissible: DismissiblePane(onDismissed: () {
                        setState(() {
                          messageData.removeAt(index);
                        });
                        ScaffoldMessenger.of(context).showSnackBar(
                            const SnackBar(content: Text('message dismissed')));
                      }),
                      children: [
                        SlidableAction( // (3)
                          onPressed: (_) {}, // (4)
                          backgroundColor: AppColor.slidableMoreColor, // (4)
                          foregroundColor: AppColor.backgroundColor, // (4)
                          icon: AppIcon.moreHorizRounded, // (4)
                          label: 'More',
                        ),
                        SlidableAction( // (3)
                          onPressed: (_) {},
                          backgroundColor: AppColor.slidableFlagColor,
                          foregroundColor: AppColor.backgroundColor,
                          icon: AppIcon.flag,
                          label: 'Flag',
                        ),
                        SlidableAction( // (3)
                          onPressed: (_) {},
                          backgroundColor: AppColor.slidableDeleteColor,
                          foregroundColor: AppColor.backgroundColor,
                          icon: AppIcon.delete,
                          label: 'Trash',
                        ),
                      ],
                    ),
                    child: Column(
```

:::details 全体のコード
```dart:app_icon.dart
import 'package:flutter/material.dart';

class AppIcon {
  static Icon arrowForwardOutlined({required Color color}) => Icon(
        Icons.arrow_forward_outlined,
        color: color,
      );

  static Icon star({required Color color, double size = 14}) => Icon(
        Icons.star,
        size: size,
        color: color,
      );

  static Icon circle({required Color color, double size = 14}) => Icon(
        Icons.circle,
        size: size,
        color: color,
      );
  static Icon arrowForwardIosOutlined(
          {required Color color, double size = 12}) =>
      Icon(
        Icons.arrow_forward_ios_outlined,
        size: size,
        color: color,
      );

  // They were used in Slidable Widget.
  static const IconData markEmailUnreadOutlined =
      Icons.mark_email_unread_outlined;
  static const IconData draftsOutlined = Icons.drafts_outlined;
  static const IconData moreHorizRounded = Icons.more_horiz_rounded;
  static const IconData flag = Icons.flag;
  static const IconData delete = Icons.delete;
}
```

```dart:app_color.dart
import 'package:flutter/material.dart';

class AppColor {
  static const mainColor = Colors.blue;
  static const appBarColor = Colors.white;
  static const backgroundColor = Colors.white;

  static const normalTextColor = Colors.black;
  static const thinTextColor = Colors.grey;

  static final iconStarColor = Colors.yellow.shade700;

  static const slidableMoreColor = Colors.grey;
  static const slidableFlagColor = Colors.orange;
  static final slidableDeleteColor = Colors.red.shade500;
}

```


```dart:main.dart
import 'package:flutter/gestures.dart';
import 'package:flutter/material.dart';
import 'package:flutter_slidable/flutter_slidable.dart';
import 'package:intl/intl.dart';
import 'package:list_slidable_app/constant/app_color.dart';
import 'package:list_slidable_app/constant/app_icon.dart';
import 'package:list_slidable_app/message_data.dart';

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
        primaryColor: AppColor.mainColor,
      ),
      home: const MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({
    Key? key,
  }) : super(key: key);

  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text(
          'Slidable App',
          style: TextStyle(
            color: AppColor.mainColor,
          ),
        ),
        actions: [
          IconButton(
              onPressed: () {},
              icon: AppIcon.arrowForwardOutlined(
                color: AppColor.mainColor,
              ))
        ],
        elevation: 0,
        backgroundColor: AppColor.appBarColor,
      ),
      body: ColoredBox(
        color: AppColor.backgroundColor,
        child: Column(
          children: [
            Padding(
              padding: const EdgeInsets.only(
                left: 32,
                top: 8,
                bottom: 8,
              ),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.start,
                children: const [
                  Text(
                    'All Inboxes',
                    style: TextStyle(
                      fontWeight: FontWeight.bold,
                      fontSize: 32,
                    ),
                  ),
                ],
              ),
            ),
            Flexible(
              child: ListView.builder(
                itemCount: messageData.length,
                itemBuilder: (BuildContext context, int index) {
                  final message = messageData[index];

                  return Slidable(
                    // enabled: false, // falseにすると文字通りスライドしなくなります
                    // closeOnScroll: false, // *2
                    dragStartBehavior: DragStartBehavior.start,
                    key: UniqueKey(),
                    startActionPane: ActionPane(
                      extentRatio: 0.2,
                      motion: const ScrollMotion(),
                      children: [
                        message['isChecked']
                            ? SlidableAction(
                                onPressed: (_) {},
                                backgroundColor: AppColor.mainColor,
                                foregroundColor: AppColor.backgroundColor,
                                icon: AppIcon.markEmailUnreadOutlined,
                                label: 'Unread',
                              )
                            : SlidableAction(
                                onPressed: (_) {},
                                backgroundColor: AppColor.mainColor,
                                foregroundColor: AppColor.backgroundColor,
                                icon: AppIcon.draftsOutlined,
                                label: 'Read',
                              )
                      ],
                    ),
                    endActionPane: ActionPane(
                      extentRatio: 0.5,
                      motion: const StretchMotion(),
                      dismissible: DismissiblePane(onDismissed: () {
                        setState(() {
                          messageData.removeAt(index);
                        });
                        ScaffoldMessenger.of(context).showSnackBar(
                            const SnackBar(content: Text('message dismissed')));
                      }),
                      children: [
                        SlidableAction(
                          onPressed: (_) {},
                          backgroundColor: AppColor.slidableMoreColor,
                          foregroundColor: AppColor.backgroundColor,
                          icon: AppIcon.moreHorizRounded,
                          label: 'More',
                        ),
                        SlidableAction(
                          onPressed: (_) {},
                          backgroundColor: AppColor.slidableFlagColor,
                          foregroundColor: AppColor.backgroundColor,
                          icon: AppIcon.flag,
                          label: 'Flag',
                        ),
                        SlidableAction(
                          onPressed: (_) {},
                          backgroundColor: AppColor.slidableDeleteColor,
                          foregroundColor: AppColor.backgroundColor,
                          icon: AppIcon.delete,
                          label: 'Trash',
                        ),
                      ],
                    ),
                    child: Column(
                      children: [
                        const Padding(
                          padding: EdgeInsets.only(left: 32),
                          child: Divider(
                            height: 1,
                            thickness: 1,
                          ),
                        ),
                        Padding(
                          padding: const EdgeInsets.all(8.0),
                          child: Column(
                            children: [
                              Row(
                                mainAxisAlignment:
                                    MainAxisAlignment.spaceBetween,
                                children: [
                                  Row(
                                    children: [
                                      // アイコンの判定---
                                      if (message['isVIP'])
                                        AppIcon.star(
                                          color: AppColor.iconStarColor,
                                        ),
                                      if (!message['isVIP'] &&
                                          !message['isChecked'])
                                        AppIcon.circle(
                                          color: AppColor.mainColor,
                                        ),
                                      if (!message['isVIP'] &&
                                          message['isChecked'])
                                        const SizedBox(
                                          width: 14,
                                        ),
                                      // ---
                                      const SizedBox(
                                        width: 8,
                                      ),
                                      Text(
                                        message['from'],
                                        style: const TextStyle(
                                          fontSize: 16,
                                          fontWeight: FontWeight.bold,
                                        ),
                                      ),
                                    ],
                                  ),
                                  Row(
                                    children: [
                                      Text(
                                        DateFormat('yyyy/MM/d')
                                            .format(message['sentAt']),
                                        style: const TextStyle(
                                          color: AppColor.thinTextColor,
                                        ),
                                      ),
                                      const Icon(
                                        Icons.arrow_forward_ios_outlined,
                                        size: 12,
                                        color: AppColor.thinTextColor,
                                      ),
                                    ],
                                  ),
                                ],
                              ),
                              Row(
                                children: [
                                  // アイコンの判定---
                                  if (message['isVIP'] && !message['isChecked'])
                                    AppIcon.circle(
                                      color: AppColor.mainColor,
                                    ),
                                  if (!message['isVIP'])
                                    const SizedBox(
                                      width: 14,
                                    ),
                                  // ---
                                  const SizedBox(
                                    width: 8,
                                  ),
                                  Text(message['title']),
                                ],
                              ),
                              Row(
                                children: [
                                  const SizedBox(
                                    width: 20,
                                  ),
                                  Expanded(
                                    child: Text(
                                      message['message'],
                                      maxLines: 2,
                                      style: const TextStyle(
                                        overflow: TextOverflow.ellipsis,
                                        color: AppColor.thinTextColor,
                                      ),
                                    ),
                                  ),
                                ],
                              ),
                            ],
                          ),
                        ),
                      ],
                    ),
                  );
                },
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

```dart:message_data.dart
List<Map<String, dynamic>> messageData = [
  {
    'isVIP': true,
    'isChecked': false,
    'from': 'Noah',
    'title': 'Hello, I\'m 平和、安らぎ!!',
    'message':
        'I watch a news, 実業家の「ひろゆき」こと、西村博之氏が１９日、ツイッターを更新。ウクライナへ千羽鶴を送る行為を「良い事をした気分になるのは恥ずかしい事」などを指摘したことに「強い言葉を使う事で、多く知れ渡り間違った行為をする人を将来的に減らせる」という考えを示した。',
    'sentAt': DateTime.now(),
  },
  {
    'isVIP': true,
    'isChecked': false,
    'from': 'Liam',
    'title': 'Hello, I\'m 勇敢な守護者!!',
    'message':
        'I watch a news, 実業家の「ひろゆき」こと、西村博之氏が１９日、ツイッターを更新。ウクライナへ千羽鶴を送る行為を「良い事をした気分になるのは恥ずかしい事」などを指摘したことに「強い言葉を使う事で、多く知れ渡り間違った行為をする人を将来的に減らせる」という考えを示した。',
    'sentAt': DateTime.now(),
  },
  {
    'isVIP': false,
    'isChecked': true,
    'from': 'Elijah',
    'title': 'Hello, I\'m 主なる神!!',
    'message':
        'I watch a news, 実業家の「ひろゆき」こと、西村博之氏が１９日、ツイッターを更新。ウクライナへ千羽鶴を送る行為を「良い事をした気分になるのは恥ずかしい事」などを指摘したことに「強い言葉を使う事で、多く知れ渡り間違った行為をする人を将来的に減らせる」という考えを示した。',
    'sentAt': DateTime.now(),
  },
  {
    'isVIP': true,
    'isChecked': false,
    'from': 'Oliver',
    'title': 'Hello, I\'m オリーブの樹（平和の象徴）!!',
    'message':
        'I watch a news, 実業家の「ひろゆき」こと、西村博之氏が１９日、ツイッターを更新。ウクライナへ千羽鶴を送る行為を「良い事をした気分になるのは恥ずかしい事」などを指摘したことに「強い言葉を使う事で、多く知れ渡り間違った行為をする人を将来的に減らせる」という考えを示した。',
    'sentAt': DateTime.now(),
  },
  {
    'isVIP': false,
    'isChecked': false,
    'from': 'James',
    'title': 'Hello, I\'m イエスの弟子!!',
    'message':
        'I watch a news, 実業家の「ひろゆき」こと、西村博之氏が１９日、ツイッターを更新。ウクライナへ千羽鶴を送る行為を「良い事をした気分になるのは恥ずかしい事」などを指摘したことに「強い言葉を使う事で、多く知れ渡り間違った行為をする人を将来的に減らせる」という考えを示した。',
    'sentAt': DateTime.now(),
  },
  {
    'isVIP': false,
    'isChecked': false,
    'from': 'Mason',
    'title': 'Hello, I\'m 石工職人!!',
    'message':
        'I watch a news, 実業家の「ひろゆき」こと、西村博之氏が１９日、ツイッターを更新。ウクライナへ千羽鶴を送る行為を「良い事をした気分になるのは恥ずかしい事」などを指摘したことに「強い言葉を使う事で、多く知れ渡り間違った行為をする人を将来的に減らせる」という考えを示した。',
    'sentAt': DateTime.now(),
  },
  {
    'isVIP': false,
    'isChecked': false,
    'from': 'Logan',
    'title': 'Hello, I\'m 小さな空洞!!',
    'message':
        'I watch a news, 実業家の「ひろゆき」こと、西村博之氏が１９日、ツイッターを更新。ウクライナへ千羽鶴を送る行為を「良い事をした気分になるのは恥ずかしい事」などを指摘したことに「強い言葉を使う事で、多く知れ渡り間違った行為をする人を将来的に減らせる」という考えを示した。',
    'sentAt': DateTime.now(),
  },
  {
    'isVIP': true,
    'isChecked': false,
    'from': 'Lucas',
    'title': 'Hello, I\'m 光を運ぶ者!!',
    'message':
        'I watch a news, 実業家の「ひろゆき」こと、西村博之氏が１９日、ツイッターを更新。ウクライナへ千羽鶴を送る行為を「良い事をした気分になるのは恥ずかしい事」などを指摘したことに「強い言葉を使う事で、多く知れ渡り間違った行為をする人を将来的に減らせる」という考えを示した。',
    'sentAt': DateTime.now(),
  },
  {
    'isVIP': false,
    'isChecked': false,
    'from': 'Mateo',
    'title': 'Hello, I\'m 神からの贈り物!!',
    'message':
        'I watch a news, 実業家の「ひろゆき」こと、西村博之氏が１９日、ツイッターを更新。ウクライナへ千羽鶴を送る行為を「良い事をした気分になるのは恥ずかしい事」などを指摘したことに「強い言葉を使う事で、多く知れ渡り間違った行為をする人を将来的に減らせる」という考えを示した。',
    'sentAt': DateTime.now(),
  },
  {
    'isVIP': false,
    'isChecked': true,
    'from': 'Ethan',
    'title': 'Hello, I\'m 堅固、長寿!!',
    'message':
        'I watch a news, 実業家の「ひろゆき」こと、西村博之氏が１９日、ツイッターを更新。ウクライナへ千羽鶴を送る行為を「良い事をした気分になるのは恥ずかしい事」などを指摘したことに「強い言葉を使う事で、多く知れ渡り間違った行為をする人を将来的に減らせる」という考えを示した。',
    'sentAt': DateTime.now(),
  },
];
```
:::

*1: 多少の誤差やボトムバーがないことなんかは気にしないでください🙇‍♂️

*2:スライドした時にそのままスライドしていくか一度しっかり止まるかの違いです。

# まとめ

いかがだったでしょうか？今回はiPhoneのメールをもとに少しネタ的な記事を書いてみました。

ぜひflutter_slidableパッケージを使ってみてください！

最後まで読んでいただきありがとうございました！

# 使用したデータ

https://ny-benricho.com/life/american-name/

https://news.yahoo.co.jp/articles/9be59f27649a402baaa2acb2d47eb2877420754f

# 参考ページ

https://stackoverflow.com/questions/55792521/how-to-fix-a-dismissed-dismissible-widget-is-still-part-of-the-tree-error-in
