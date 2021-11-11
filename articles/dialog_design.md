---
title: "【Flutter】コピペで使えるシンプルなダイアログ" # 記事のタイトル
emoji: "🐶" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Dart"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに
ご覧いただきありがとうございます。ganです。

今回はFlutterでアプリを開発する際におそらく誰もが使うであろうダイアログについて書きました。

最低限のデザインと汎用性を重視して作りました。

読んでくださった方のお力になれば幸いです。

# ダイアログとは

ダイアログって何？どんな種類があるの？と思った方、こちらの記事が参考になると思います。

https://qiita.com/coka__01/items/4c1aea5fb1646e463f91#cupertino%E7%B7%A8

ちなみに今回、自分はMaterialなダイアログを作りました。


# 完成品

## はい/いいえ が選択できるバージョン

![](https://storage.googleapis.com/zenn-user-upload/aa3ef3290b73d80c6ffef03e.png)


## OK のみのバージョン

![](https://storage.googleapis.com/zenn-user-upload/2d7fd48a9a649fcf44a2c843.png)


# 完成品のコード

はじめに自身でdartのファイルを作成してそこにコピペして使ってください。


```dart:original_dialog.dart
import 'package:flutter/material.dart';

/// はい/いいえ が選択できるバージョン
Future showConfirmDialog(
  context, {
  required String title,
  required String content,
  required onApproved,
}) async {
  showDialog(
      context: context,
      // barrierDismissibleのbool値をtrueにすると、
      // ダイアログの外側を押した際にダイアログが出る前の画面に戻れるようになります。
      barrierDismissible: false,
      builder: (BuildContext context) {
        return Dialog(
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(10.0),
          ),
          child: Container(
            // デバイスに応じて横幅(width)は調整してください。
            width: 311.0,
            decoration: BoxDecoration(
              border: Border.all(color: Colors.blueAccent, width: 3),
              borderRadius: BorderRadius.circular(10.0),
            ),
            child: Column(
              // mainAxisSize: MainAxisSize.min があることで、
              // Columnの子要素の縦幅に合わせて表示できます。
              mainAxisSize: MainAxisSize.min,
              children: <Widget>[
                Padding(
                    padding: const EdgeInsets.symmetric(vertical: 24.0),
                    child: Text(
                      title,
                      style: const TextStyle(
                        fontSize: 18.0,
                        fontWeight: FontWeight.bold,
                      ),
                    )),
                Padding(
                  padding: const EdgeInsets.symmetric(horizontal: 24),
                  child: Text(
                    content,
                  ),
                ),
                const SizedBox(
                  height: 24.0,
                ),
                Row(
                  mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                  children: [
                    ElevatedButton(
                      style: ElevatedButton.styleFrom(
                        side: const BorderSide(
                          width: 1.0,
                          color: Colors.blueAccent,
                        ),
                        shadowColor: Colors.grey,
                        // elevation で影の長さを指定
                        elevation: 5,
                        // primary でボタンの背景色を指定
                        primary: Colors.white,
                        // onPrimary でボタンないの文字の色を指定
                        onPrimary: Colors.black,
                        // shape: const StadiumBorder() でボタンのサイドがまるくなります。
                        shape: const StadiumBorder(),
                      ),
                      onPressed: onApproved,
                      child: const Padding(
                        padding:
                            EdgeInsets.symmetric(vertical: 16, horizontal: 18),
                        child: Text('いいえ'),
                      ),
                    ),
                    ElevatedButton(
                      style: ElevatedButton.styleFrom(
                        shadowColor: Colors.grey,
                        elevation: 5,
                        primary: Colors.blueAccent,
                        onPrimary: Colors.white,
                        shape: const StadiumBorder(),
                      ),
                      onPressed: () {
                        Navigator.of(context).pop();
                      },
                      child: const Padding(
                        padding:
                            EdgeInsets.symmetric(vertical: 16, horizontal: 18),
                        child: Text('はい'),
                      ),
                    ),
                  ],
                ),
                const SizedBox(
                  height: 24.0,
                ),
              ],
            ),
          ),
        );
      });
}
```

呼び出す際は、

```dart
onPressed: () {
          showConfirmDialog(
            context,
            // 好きな文字列を入れてください。
            title: 'タイトル',
            // 好きな文字列を入れてください。
            content: 'テキストテキストテキストテキストテキストテキストテキストテキスト',
            onApproved: () {
              // はい が押された時の処理を入れる。
              // 以下は例
              Navigator.of(context).pop();
            },
          );
        },
```

このようにすると良いです。

OK のみのバージョンも書いておきます。使い方は上とほとんど同じです。


```dart
/// OK のみのバージョン
showAlertDialog(
  context, {
  required String title,
  required String content,
}) async {
  showDialog(
      context: context,
      barrierDismissible: true,
      builder: (BuildContext context) {
        return Dialog(
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(10.0),
          ),
          child: Container(
            width: 311.0,
            decoration: BoxDecoration(
              border: Border.all(color: Colors.blueAccent, width: 3),
              borderRadius: BorderRadius.circular(10.0),
            ),
            child: Column(
              mainAxisSize: MainAxisSize.min,
              children: <Widget>[
                Padding(
                    padding: const EdgeInsets.symmetric(vertical: 24.0),
                    child: Text(
                      title,
                      style: const TextStyle(
                        fontSize: 18.0,
                        fontWeight: FontWeight.bold,
                      ),
                    )),
                Padding(
                  padding: const EdgeInsets.symmetric(horizontal: 24),
                  child: Text(
                    content,
                  ),
                ),
                const SizedBox(
                  height: 24.0,
                ),
                Row(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    ElevatedButton(
                      style: ElevatedButton.styleFrom(
                        shadowColor: Colors.grey,
                        elevation: 5,
                        primary: Colors.blueAccent,
                        onPrimary: Colors.white,
                        shape: const StadiumBorder(),
                      ),
                      onPressed: () {
                        Navigator.of(context).pop();
                      },
                      child: const Padding(
                        padding:
                            EdgeInsets.symmetric(vertical: 16, horizontal: 36),
                        child: Text('OK'),
                      ),
                    ),
                  ],
                ),
                const SizedBox(
                  height: 24.0,
                ),
              ],
            ),
          ),
        );
      });
}
```

### 注意

Flutter2.0以降でない場合は``require``を``@require``に変更してください。


# 終わりに

ダイアログはどんなアプリであっても必要になります。

サクッと用意しておきたい！と思ったらぜひ参考にしてください。

