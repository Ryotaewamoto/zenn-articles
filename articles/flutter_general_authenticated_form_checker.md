---
title: "【Flutter】 新規登録における入力フォームの実装で考慮すべき点とその対応" # 記事のタイトル
emoji: "👶" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Dart"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

[2023 Flutter大学アドベントカレンダー5日目](https://qiita.com/advent-calendar/2023/flutteruniv
)の記事になります。

ご覧いただきありがとうございます。[gan](https://zenn.dev/ryota_iwamoto)です。今回は **【Flutter】 認証周りの実装時に考慮すべき点とその対応** ということで、Flutter を使用してモバイルアプリを開発する際の認証周りの実装時、特に入力フォーム周辺で、これは気をつけた方がいい思ったことをまとめました。実装後の確認としてもこの記事が役に立てば幸いです。

## 内容

新規登録の実装時に考慮すべき点とその対応

1. フォーカスが当たった時のフォーム挙動
2. フォーカスを外す処理の追加
3. キーボードの指定

## 新規登録の実装時に考慮すべき点とその対応

新規登録はそのアプリの入り口にあたる部分であり、その体験を向上させることはユーザを確保するための重要な要素の一つです。ここで、ユーザの登録には一般的に、メールアドレスとパスワードを使用するものとソーシャルログイン (SNS アカウントや GitHub アカウントを使用したログイン方法) を使用するものの２つがあります。
ここでは、メールアドレスとパスワードを入力するフォームを実装することを想定して進めていきます。

### 1. フォーカスが当たった時のフォーム挙動

ユーザは、入力フォームを押した際、画面上にはキーボードが表示され入力待機状態になったことを確認できます。しかし、いくらカーソルがあるとはいえ、複数ある入力フォームの内のどこにフォーカスが当たっているのかはわかりません。そのため、**フォーカスが当たった時に、その入力フォームの枠の色を変更するなどの視覚的な変化を与える**ことで、ユーザがどの入力フォームにフォーカスが当たっているのかをわかりやすくすることができます。

一般に、

```dart
TextFormField(
  // ...
  decoration: InputDecoration(
    focusedBorder: // ...,
    errorBorder: // ...,
  ),
  // ...
)
```

と各入力フォームで定義されることが多いです。この実装で上に記述したような懸念点の対応は十分できています。しかしながら、このような書き方の場合、各入力フォームで同じようなコードを書くことになります。これは、コードの重複を招くことになり、メンテナンス性が低下する原因となります。そこで、このようなケースでは、`MaterialApp` の `ThemeData` に `InputDecorationTheme` を追加することで、コードの重複を防ぐことができます。

```dart
return MaterialApp(
      theme: ThemeData(
        // ...,
        inputDecorationTheme: const InputDecorationTheme(
          contentPadding: EdgeInsets.all(16),
          focusedBorder: TextFormBorders.textFormFocusedBorder,
          enabledBorder: TextFormBorders.textFormEnabledBorder,
          focusedErrorBorder: TextFormBorders.textFormErrorBorder,
          errorBorder: TextFormBorders.textFormErrorBorder,
        ),
      ),
  );
```

:::details OutlineInputBorder を定義した TextFormBorders クラス

```dart
class TextFormBorders {
  // キーボード表示時のフォームの枠線。
  static const textFormFocusedBorder = OutlineInputBorder(
    borderRadius: BorderRadius.all(Radius.circular(8)),
    borderSide: BorderSide(
      color: Colors.deepPurple,
      width: 2,
    ),
  );

  // 平常時のフォームの枠線。
  static const textFormEnabledBorder = OutlineInputBorder(
    borderRadius: BorderRadius.all(Radius.circular(8)),
    borderSide: BorderSide(
      color: Colors.grey,
    ),
  );

  // 入力中のエラー時の枠線。
  static const textFormErrorBorder = OutlineInputBorder(
    borderRadius: BorderRadius.all(Radius.circular(8)),
    borderSide: BorderSide(
      color: Colors.red,
      width: 2,
    ),
  );
}
```

:::

これにより、各入力フォームで重複したコードを書くことなく、フォーカスが当たった時のフォームの枠の色を変更することができます。

| 通常時 | フォーカス時 |
| ---- | ---- |
| ![normal](https://storage.googleapis.com/zenn-user-upload/523f6fc97a7d-20231208.png) | ![focusing](https://storage.googleapis.com/zenn-user-upload/a8f86e66582a-20231208.png) |

### 2. フォーカスを外す処理の追加

ユーザは、フォームの入力完了後にフォーカスが外れること、つまり、キーボードが閉じられることを期待しています。そのため、フォーカスが外れた時に、前述したような入力フォームの枠の色を変更するなどの視覚的な変化を与えることで、ユーザがフォームの入力が完了したことをわかりやすくすることができます。この時、特に、**入力フォーム以外をタップすることでフォーカスが外れる実装**をすべきです。そのためには、各入力フォームにおいて、

```dart
TextFormField(
  // ...
  onTapOutside: (_) => FocusManager.instance.primaryFocus?.unfocus(),
  // ...
)
```

とするのが望ましいです。

:::message alert
ただし、この実装は Flutter のバージョンが **3.7.0 以上**である必要があります。それ以前のバージョンでは、`onTapOutside` は使用できませんので、代わりに `GestureDetector` を使用して、入力フォーム以外をタップした際にフォーカスが外れるようにする必要があります。
:::

また、`TextFormField` の `onEditingComplete` にも同様の対応をすることで、キーボードの完了ボタンを押した際にもフォーカスが外れるようにすることができます。

```dart
TextFormField(
  // ...
  onEditingComplete: () => FocusManager.instance.primaryFocus?.unfocus(),
  // ...
)
```

### 3. 入力値のバリデーションとキーボードの指定

メールアドレスやパスワードの入力フォームにおいて、**入力値のバリデーション**と**キーボードの指定**を行うことは非常に重要です。これは、入力フォームにおいて、正しい形式でない文字列が入力された場合、その入力フォームの下にエラーメッセージを表示することで、ユーザに入力値が正しい形式でないことを伝えることができます。また、キーボードの指定については、入力内容に応じたキーボードの種類を指定することで、ユーザが入力しやすいようにすることができます。事前にメールアドレス、パスワードの正規表現によるバリデーションを用意しておきます。(正規表現についてはこの記事では詳しく説明しません。各自で適切な正規表現を使用してください。)

| メールアドレスの場合 | パスワードの場合 |
| ---- | ---- |
| ![mail](https://storage.googleapis.com/zenn-user-upload/c8fb5ad02107-20231208.png) | ![password](https://storage.googleapis.com/zenn-user-upload/fd5778b0a810-20231208.png) |

```dart
extension StringEx on String {
  /// メールアドレスの正規表現バリデーション
  bool isValidEmail() {
    return RegExp(
      r'^(([^<>()[\]\\.,;:\s@\"]+(\.[^<>()[\]\\.,;:\s@\"]+)*)|(\".+\"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$',
    ).hasMatch(this);
  }

  /// パスワードの正規表現バリデーション
  /// 8文字以上、大文字・小文字・数字をそれぞれ1文字以上含む必要がある。
  bool isValidPassword() {
    return RegExp(
      r'^(?=.*?[a-z])(?=.*?[A-Z])(?=.*?[0-9]).{8,100}$',
    ).hasMatch(this);
  }
}
```

#### メールアドレスの場合

こちらはメールアドレスの場合の例です。`inputFormatters` を使用して入力できる文字を制限しています。また、`keyboardType` を使用してキーボードの種類を指定しています。`errorText` には、入力値が正しい形式でない場合に表示するエラーメッセージを設定しています。

```dart
TextFormField(
  // ...
  keyboardType: TextInputType.emailAddress,
  inputFormatters: [
    FilteringTextInputFormatter.allow(RegExp(r'[a-zA-Z0-9@.+_-]')),
  ],
  errorText: _errorText,
  // ...
)
```

```dart
String? get _errorText {
  // 入力されたテキストを取得する。
  final text = controller.value.text;
  if (text.isNotEmpty && !text.isValidEmail()) {
    return '正しい形式で入力してください。';
  }
  // エラーがない場合は null を返す。
  return null;
}
```

| keyboardType 指定無し | keyboardType 指定有り |
| ---- | ---- |
| ![normal](https://storage.googleapis.com/zenn-user-upload/a8f86e66582a-20231208.png) | ![specific_keyboard_type](https://storage.googleapis.com/zenn-user-upload/3dae61cdda5c-20231208.png) |

#### パスワードの場合

こちらはパスワードの例です。メールアドレスの場合に対して、`keyboardType` に `TextInputType.visiblePassword` を指定し、`inputFormatters` で同様に入力できる文字を制限しています。こちらは、大文字、小文字、数字、記号を入力できるようにしています。`errorText` には、入力値が正しい形式でない場合に表示するエラーメッセージを設定しています。

```dart
TextFormField(
  // ...
  keyboardType: TextInputType.visiblePassword,
  inputFormatters: [
    FilteringTextInputFormatter.allow(RegExp(r'[a-zA-Z0-9!-/:-@[-`{-~]')),
  ],
  errorText: _errorText,
  // ...
)
```

```dart
String? get _errorText {
  // 入力されたテキストを取得する。
  final text = controller.value.text;
  if (text.isNotEmpty && !text.isValidPassword()) {
    return '大文字、小文字、数字を含む8文字以上';
  }
  // エラーがない場合は null を返す。
  return null;
}
```

ここで一つ面白いのが、`keyboardType: TextInputType.visiblePassword,` としていますが実は、**キーボードのタイプはこれを指定するかどうかに寄りません**。キーボードのタイプは `TextFormField` の `isObscure` の指定よって決まります。`isObscure` は、入力された文字を隠すかどうかを決めるものです。`true` にすると、入力された文字が隠され、`false` にすると、入力された文字は隠されません。この `isObscure` は、パスワードの入力フォームにおいて、入力された文字を隠すために使用されます。そのため、`keyboardType` に `TextInputType.visiblePassword` を指定しているかどうかに関わらず、`isObscure` が `true` になっている場合は、キーボードのタイプは `TextInputType.visiblePassword` になります。逆に、`isObscure` が `false` になっている場合は、キーボードのタイプは `TextInputType.text` になります。

| isObscure が false | isObscure が true |
| ---- | ---- |
| ![isObscure_is_false](https://storage.googleapis.com/zenn-user-upload/f51ee369a493-20231208.png) | ![isObscure_is_true](https://storage.googleapis.com/zenn-user-upload/3ddd54f2ef93-20231208.png) |

:::message
`errorText` を表示する際、`TextFormField` の下部にスペースがない場合、その `errorText` が作るスペースが入力フォームの高さを変更してしまいます。そのため、そのような高さの変更を打ち消す際には、`TextFormField` を `SizedBox` で覆い、高さとして 104 程度を指定してあげることで解決できます。
:::

## まとめ

いかがだったでしょうか。ユーザ登録はアプリのユーザ数を増やす上で最初の大きな壁です。ぜひこの記事を参考に、またさらに細部までこだわることでユーザ登録の体験を向上させていきましょう！
