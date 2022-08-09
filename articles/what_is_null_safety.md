---
title: "【Dart】Null Safety って何の意味があるの？" # 記事のタイトル
emoji: "🤨" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Dart"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに
ご覧いただきありがとうございます。[gan](https://zenn.dev/ryota_iwamoto)です。

この記事ではDart のNull Safety について**何がどういいのか**を解説していきます。

正直、これまで自分はNull Safetyに関して、**書き方**は知っているがその**意味**に関しては全くわかりませんでした。
もしかしたら自分以外にも何気なくNull Safety を使っている人はいるかもしれません。（正確にはSound Null Safety を備えた言語を使っている、ですね）

**「Null Safety って何の意味があるの？」**

、これは私の純粋な疑問です。
もしこれを聞かれた時にサクッと答えられたら、シンプルにカッコ良くないですか？？
それにコードが書けるエンジニアはたくさんいます。ですが一流のエンジニアを目指すのであればこういう言葉の意味までよく理解しておく必要があると私は思います。
ということでこの記事を読んでNull Safety の理解を深めましょう！

:::message
使い方に関してはさまざまな良い記事があるのそちらを読んでください。ここではNull Safety とは何かをまとめた上で、その意味についてより理解を深めていただければと思います。
:::


# 用語の解説

ます初めにいくつかよく出てくる言葉の解説します。

## Sound Null Safety

公式サイトでは冒頭で、

> The Dart language now supports sound null safety!

と記されています。ここから**Sound Null Safety とは、Dart という言語に備わっている機能**のことだとわかります。ここからSound Null Safety は、あくまでシステムを指しています。

## Null Safety

Null Safety を日本語で訳すとすれば「Null安全（もしくはNull安全性）」でしょう。
ではNull Safety とSound Null Safety の違いは何でしょうか？

Sound の意味を考慮するのであれば、Sound Null Safety はNull Safety を**音に出すor警告する**ものということになります。

公式でもSound Null Safety という言葉を使っているのは冒頭程度で、それ以外はNull Safety という言葉を使っています。そのため、説明などをする際にはNull Safety という言葉を使っていくのがよさそうです。

:::message alert
Null Safety の説明の際に、

Sound の意味を考慮するのであれば、Sound Null Safety はNull Safety を音に出すor警告するもの

と書きましたがこれは間違いです。ここでのSound は音を出すといった動詞の意味ではなく、**健全な**という形容詞的な意味になります。
よって、Sound Null Safety の意味は**健全なNullの安全性**ということになります。

[kariya1975](https://zenn.dev/kariya1975)さん、教えていただきありがとうございます。

(2022/08/09 追記)
:::


## Null許容型

漢字からもわかるように、Nullも許す型という意味です。これは皆さんもすでに使っているものだと思いますので、説明は詳しくしませんが、型名の末尾に``?``をつけることでNull許容型にすることができます。

例：

```dart

class User {
    String? name;
    int? age;
}

```


# Null Safety の意味


こちらがこの記事の本題になります。意味というと難しいので、**これがNull Safety のメリット**というのを考えます。

つまりDart にSound Null Safety がデフォルトで付いているメリットを一言で言えば、それは

**Null許容型の変数のNull という状態に対してコンパイルエラーを出してくれる**

ということだと私は考えます。コンパイルエラー、つまりは実行時ではなく**コードを書いている段階で**

「Null によるエラーが発生し得るで〜」

と、警告してくれているのです。少しDart を使った例を見てみましょう。

## Sound Null Safety がないDart

まず**Sound Null Safety がないDart の場合** を見てください。

```dart

class User {
    String firstName;
}

```

シンプルなクラスですね。Flutterを使って、``firstName``をどこかアプリの画面で表示したい！と思ったときに、

```dart

User user = User();

///...

child: Text(user.firstName),

```

このように書いたとします。この場合、Sound Null Safety がないDart ではコンパイルエラーにはならずに実行時にエラーとしてログが出ます。
``User``クラスを見たらわかるように``String``型である``fisrtName``はコンストラクタによって値が代入されていません。つまりNullの状態です。にもかかわらずTextの引数の中で呼ばれているためにエラーが発生します。

## Sound Null Safety があるDart

次に**Sound Null Safety があるDart の場合** を見てください。

```dart
class User {
    String firstName; // コンパイルエラー
}
```

とした場合、この時点でエラーが出ます。定義した段階ではこれはNull の状態です。これに対してNull許容型で定義した``User``クラスは以下のようになります。

:::message
適切にコンストラクタを定義すればNull許容型にしなくても大丈夫ですが、本題から逸れるためここでは割愛します。
:::

```dart
class User {
    String? firstName;
}
```

Dart では、型の末尾に``?``にをつけることでnull許容型にすることができます。そして先ほどと同様にどこかで呼び出してみましょう。Null Safety でない場合と同じようにして書くとこれはエラーになります。

```dart

User user = User();

///...

child: Text(user.firstName), // コンパイルエラー

```

これでは、``firstName``はNull になり得るからダメだよ！Text の引数にはString型を渡してね！と注意されます。

以下のようにすればエラーの文言は消えますが、結局``User``クラスの``firstName``はnullなので実行時にエラーが出ます。Null Safety があるとは言え、完璧にエラーをなくすにはどちらにしろ書き手の意識が必要があるということです。

```dart

User user = User();

///...

child: Text(user.firstName!),　// 「!」を加えた

```

[monoさんの記事](https://medium.com/flutter-jp/null-safety-fe6503a81d5c)でも冒頭に、

> まず大事なのは、! (非nullであることのassert)を、過不足なく使うことです。無闇に乱用するのも、逆に過度に避けすぎるのもどちらも良くないです。

というように書かれています。こちらの記事は例も豊富なので読んでおくことをお勧めします。

## 二つの違いのまとめ

おそらく上の例であれば仮にNull Safetyがなくとも、エラーが出てもすぐに対処できるでしょう。ですがコードが増えていった場合はどうでしょう？毎回実行するたびにエラーが出るのはめんどくさいですし、作業の質も下がってくると思います。Sound Null Safety があれば実行せずともエラーがわかるので、やはりNull Safety のメリットは、

**Null許容型の変数のNull という状態に対してコンパイルエラーを出してくれる**

ということではないかと思います。Null Safety というのは絶対的に必要なものではなく、Nullによるエラーを言語の設定として防ぐものであるということです。


# まとめ

**Null Safety って何の意味があるの？**

そう思ったのでこの記事を書いてみました。
おそらく上の例以外にもNull Safety だと良い事例はいくつもあると思います。そのような例があれば是非とも教えてほしいです！

疑問に思ったことはこれからもアウトプットしていこうともいます！

よろしくお願いします！！

# 参考文献

https://dart.dev/null-safety

https://zenn.dev/kboy/articles/ae607839cd4573

https://medium.com/flutter-jp/null-safety-fe6503a81d5c

- [What does "sound" mean in dart "sound null safety" feature?](https://stackoverflow.com/questions/70876354/what-does-sound-mean-in-dart-sound-null-safety-feature#:~:text=Sound%20null%20safety%20makes%20types,at%20compile%2Dtime%20and%20fixed.)
