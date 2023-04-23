---
title: "【Flutter】初めてのRiverpod+Hooks+Freezed+Firebase" # 記事のタイトル
emoji: "🔥" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Dart", "Firebase"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに
ご覧いただきありがとうございます。[gan](https://zenn.dev/ryota_iwamoto)です。

最近のFlutter情報をキャッチアップしていると、Riverpod、Hooks、Freezedというのをよく聞きます。なので今回はそれらを使ったTODOアプリを作ってみて勉強になったこと、気になったところを紹介していきます。

アプリは以下の動画を真似て作りました。勉強のために少し変えている部分もあります。

https://www.youtube.com/watch?v=vrPk6LB9bjo

最終的には以下のようなアプリになります。

![](https://storage.googleapis.com/zenn-user-upload/fb409d8bdd7b-20220417.gif)

# 対象者
- Providerはある程度触ったことがある
- Firebaseを使ったことがある
- Riverpodを聞いたことはあるけど、まだそんなに触ったことがない
- Freezed？？🥶
- Flutter Hooks？Hooks？？

# 開発環境
MacBook Pro 2020 (Intel)
Flutter: 2.10.3 • channel stable
Dart: 2.16.1 • DevTools 2.9.2
IDE: VSCode


# 今回の内容
1. Riverpodとは
2. Freezedとは
3. Flutter Hooksとは
4. 実際にFirebaseも使って買い物リストアプリを作ってみる

# 1.Riverpodとは
RiverpodはDart, Flutterで使用できる「状態管理ライブラリ」です。「状態管理ライブラリ」にはProviderやBloc、GetXなどがあります。

このProviderという状態管理方法にはいくつか課題があります。

その課題とは、
- 範囲外アクセスなどで、コンパイルエラーではなく実行時例外”ProviderNotFoundException”が発生する
- 同じ型のproviderを複数同時に利用することができない
- ChangeNotifierProviderなど「View」と「状態＋ロジック」は分離できるが、「状態」と「ロジック」の分離ができない（限られた範囲でProviderを使用している場合に画面遷移をしたりするとProviderが破棄されてしまう）
などです。

これらの課題を克服するために作られたのが**Riverpod**です。

今回はこのRiverpodを使っていきます。

## Provider vs Riverpod
ProviderとRiverpod、どちらを使うかはかなり好みになるかと思います。初心者であればProviderのChangeNotifierProviderを使うのが分かりやすくていいと思います。いきなりRiverpodから入るとただ若干複雑なので一度Providerを触ってからでもいいと思います。またRiverpod自体新しく、バージョンの変更によって他の記事を見てやってもうまくいかないみたいなことが予想されるのも理由のひとつです。[こんぶさん]([https://zenn.dev/pressedkonbu](https://zenn.dev/pressedkonbu))の記事

https://zenn.dev/pressedkonbu/articles/stateful-is-good

にはかなり同意です。自分はProviderから入ったのですが、RiverpodからやっていたらFlutter引退してたかも。

個人的にはある程度状態管理を理解して、機能が多くなりそうなアプリを作るということであればRiverpodでもいいのかなと思います。Riverpodを書いてみて思ったのは状態管理のしやすさです。Freezedと組み合わせることでかなりmodel(entityもしくはdomainのこと)の操作がわかりやすくなります。（Provider＋StateNotifier＋Freezedで近いことはできます。）

**具体的な使い方についてはこの後実際のコードをみて解説します。**

### 参考サイト
https://zenn.dev/riscait/books/flutter-riverpod-practical-introduction/viewer/what-is-riverpod
https://qiita.com/datake914/items/f91acf30a640447c57c8#%E5%95%8F%E9%A1%8C%E7%82%B9-2
https://medium.com/flutter-jp/state-1daa7fd66b94

# 2.Freezedとは
Freezedに入る前に**mutable**と**immutable**の違いについて書いておきます。

## mutableとimmutable
インスタンスにはmutable（可変）、immutable（不変）という考え方があります。もしかしたらどこかで単語は聞いたことがあるかも？

可変や不変と言われてもよくわからないと思うので最低限のコードを使って考えてみます。Personクラスを作ってその中のnameプロパティとageプロパティを変更してみます。

- mutableの場合
```dart
void main() {
  createMutablePerson();
}

class Person {
  Person(this.name, this.age);
  String name;
  int age;
}

void createMutablePerson() {
  // インスタンスを生成
  final personA = Person('Kenta', 30);
  // この場合Personクラスのnameとageは再代入可能なので値を変更できる
  personA.name = 'Daigo';
  personA.age++;
  print(personA.name); // Daigo
  print(personA.age); // 31
}
```

- immutableの場合
```dart
void main() {
  createImmutablePerson();
}

class Person {
  Person(this.name, this.age);
  final String name;
  final int age;
}

void createImmutablePerson() {
  // インスタンスを生成
  final personA = Person('Kenta', 30);
  // この場合Personクラスのnameとageはfinalが付いているので再代入不可、よって値を変更できない
  personA.name = 'Daigo';
  personA.age++;
  print(personA.name); // エラー
  print(personA.age); // エラー
}
```

class Personのひとつ前の行に@immutableというアノテーションをつけることでimmutableを強制させることもできます。

またimmutableにした際に、インスタンスをコピーして使いたいのでcopyWithを定義する必要が出てきます。ここで登場するのがFreezedです。Freezedを使うことでcopyWith以外にもtoStringや==についても自動生成してくれています。（この辺りはまだ自分もよくわかっていない）

## Freezedの使い方
大まかな使い方は[さくしんさん](https://zenn.dev/sakusin)の記事
https://zenn.dev/sakusin/articles/b19e9a2c3829e0
がかなり参考になります。公式のYouTube動画は短いのでちょっと見てみるだけでもかなり参考になると思います。
https://www.youtube.com/watch?v=RaThk0fiphA

### 参考サイト
https://medium.com/flutter-jp/immutable-d23bae5c29f8
https://zenn.dev/purigen/articles/90860436b855a2872b10

# 3.Flutter Hooksとは

自分はHooksの存在を最近知りました。以下の動画が分かりやすかったのでぜひ時間がある時に見てみるといいと思います。

https://www.youtube.com/watch?v=HZD8IiWZ7Ac

一言で言うと、React HooksのFlutter版で文法や使い方が同じもの、だそうです。

、、、いや、わからん。モバイルから入った人(自分のような)からしたらまずReactって何なりますよねー。

なのでHooksの歴史や起源は置いておいて簡単な使い方だけまとめておきます。

## 使い方

:::message
少し説明が複雑にはなりますが、Flutter Hooksを使う際のRiverpodの使い方についても最後に書いておきます。
:::

RiverpodとFlutter Hooksをインストール

```yaml
environment:
  sdk: ">=2.7.0 <3.0.0"
  flutter: ">= 1.17.0"

dependencies:
  flutter:
    sdk: flutter
      flutter_hooks: ^0.18.3
  hooks_riverpod: ^1.0.3 // 純粋なriverpodじゃないので注意
```

ターミナルでflutter pub getを実行

```terminal
$ flutter pub get
```

以下にサンプルとしてカウンターアプリのコードを載せておきます。
これがHooksの基本的な使い方になります。

```dart
import 'package:flutter/material.dart';
import 'package:flutter_hooks/flutter_hooks.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';

final counterProvider = StateNotifierProvider((_) => Counter());

class Counter extends StateNotifier<int> {
  Counter() : super(0);
  void increment() => state++;
}

void main() {
  runApp(
    const ProviderScope(
      child: CounterApp(),
    ),
  );
}

class CounterApp extends HookConsumerWidget {
  const CounterApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch(counterProvider);
    final isWhiteBackGround = useState(true);

    return MaterialApp(
      home: Scaffold(
      backgroundColor: isWhiteBackGround.value ? Colors.white : Colors.black,
        appBar: AppBar(
          title: const Text('CounterApp'),
          leading: IconButton(
            icon: const Icon(Icons.settings),
            onPressed: () => isWhiteBackGround.value = !isWhiteBackGround.value,
          ),
        ),
        body: Center(
          child: Text(
            state.toString(),
            style: TextStyle(
              fontSize: 40,
              fontWeight: FontWeight.bold,
              color: isWhiteBackGround.value ? Colors.black : Colors.white
            ),
          ),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () => ref.read(counterProvider.notifier).increment(),
          child: const Icon(Icons.add),
        ),
      ),
    );
  }
}
```

### サンプルコード解説

まずRiverpodを使うのでCounterAppウィジェットを``ProviderScope``でラップしてください

```dart
void main() {
  runApp(
    const ProviderScope(
      child: CounterApp(),
    ),
  );
}
```

次にFlutter Hooksを使いたいクラスに``HookConsumerWidget``を継承してください。

これはHookWidgetとConsumerWidgetの合体Widgetだと思ってもらって差し支えないです。

```dart
class CounterApp extends HookConsumerWidget {
  const CounterApp({Key? key}) : super(key: key);

```

そしてbuild関数内にuseStateを使って値を変化させたいものを定義します。

上のサイバーエージェントの動画ではint型を使っているのでここではbool型を使ってみます。引数には初期値を代入してください。

:::message alert
HookConsumerWidgetを継承している場合、build関数の第二引数はWidgetRefになるので注意です。

```dart
@override
  Widget build(BuildContext context, WidgetRef ref) {
    final isWhiteBackGround = useState(true);
```
:::

最後にWidgetを上のようなコードにすれば、左上の設定ボタンを押すと背景と文字の色を変化させることができます。

### 完成品(Flutter Hooksのみ)
![](https://storage.googleapis.com/zenn-user-upload/be4c8a7106e6-20220417.gif)

次にRiverpodの使い方です。

まずはStateNotifierを継承したCounterクラスを作成します。

StateNotifier<T>のTの部分にstateの型をいれます。これが監視対象です。

super(0)とすることでCounterのstateに0を代入しています。

increment()を呼ぶことでstateの値を+1します。

そしたらファイル内に以下のを書いておきます。StateNotifierProviderを使用します。

```dart
final counterProvider = StateNotifierProvider((ref) => Counter());
```

最後にHookConsumerWidgetを継承しているクラス内のbuild関数内で以下のように定義します。

```dart
final state = ref.watch(counterProvider);
```

これでCounterクラスのstateをWidgetツリーの中で使うことができます。

```dart
        body: Center(
          child: Text(
            state.toString(),// ココ！
            style: TextStyle(
              fontSize: 40,
              fontWeight: FontWeight.bold,
              color: isWhiteBackGround.value ? Colors.black : Colors.white
            ),
          ),
        ),
```

### 完成品(Riverpod+Flutter Hooks)

![](https://storage.googleapis.com/zenn-user-upload/6f1da244f55c-20220417.gif)

:::message alert
他の少し古い記事を見てみるとuseProvider()を使っていることが多いと思います。
useProviderはRiverpodのアップデートで使用できなくなっているので基本はref.watch()を使います。ただしIconButtonのonPressed内でなど非同期な処理を行う場合はref.read()を使用してください。
:::

Riverpodのバージョンによる変更は以下の記事がとても参考になります！

https://zenn.dev/riscait/books/flutter-riverpod-practical-introduction/viewer/migrate-to-v1

### 参考サイト
https://qiita.com/karamage/items/8d1352e5a4f1b079210b#riverpod-%E3%81%A8-flutter-hooks-%E3%81%A8-statenotifier%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB

# 4.実際にFirebaseも使って買い物リストアプリを作ってみる

## ファイル構成

> lib/
　├ controllers/
　│　└ auth_controller.dart
　│　└ item_list_controller.dart
　├ extensions/
　│　└ firebase_firestore_extension.dart
　├ models/
　│　└ item_model.dart
　│　└ item_model.freezed.dart
　│　└ item_model.g.dart
　├ repositories/
　│　└ auth_repository.dart
　│　└ custom_exception.dart
　│　└ item_repository.dart
　├general_provider
　└ main.dart

GitHubのリポジトリのリンクを貼っておくので興味がある人は見てみてください。

https://github.com/Ryotaewamoto/riverpod_pracitce_app

ここではFirebaseに絞って解説します。おそらくここが一番気になる人が多いと思うので！

## Firebase
FirebaseについてはFirestoreとFirebase Authを使います。ここでは認証として「匿名」を使用しています。

(追記:2022/07/09)
メールアドレスを用いた認証に関しては別の記事で書いたのでよければ参考にしてください！

https://zenn.dev/ryota_iwamoto/articles/firebase_auth_and_riverpod


FirestoreのCRUD操作についてはRiverpodを用いて以下のように記述しています。
また抽象クラスを作る意味については以下の2つの記事が参考になると思います。

https://qiita.com/tokkun5552/items/4649478803be47cd488c

https://qiita.com/aiko_han/items/e8ddce85188970fd77da

```dart:item_repository.dart
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';
import 'package:riverpod_practice_app/extensions/firebase_firestore_extension.dart';
import 'package:riverpod_practice_app/general_provider.dart';
import 'package:riverpod_practice_app/models/item_model.dart';
import 'package:riverpod_practice_app/repositories/custom_exception.dart';

abstract class BaseItemRepository {
  Future<List<Item>> retrieveItem({required String userId});
  Future<String> createItem({required String userId, required Item item});
  Future<void> updateItem({required String userId, required Item item});
  Future<void> deleteItem({required String userId, required String itemId});
}

final itemRepositoryProvider =
    Provider<ItemRepository>((ref) => ItemRepository(ref.read));

class ItemRepository implements BaseItemRepository {
  final Reader _read;

  const ItemRepository(this._read);

  @override
  Future<List<Item>> retrieveItem({required String userId}) async {
    try {
      final snap =
          await _read(firebaseFirestoreProvider).userListRef(userId).get();
      return snap.docs.map((doc) => Item.fromDocument(doc)).toList();
    } on FirebaseException catch (e) {
      throw CustomException(message: e.message);
    }
  }

  @override
  Future<String> createItem(
      {required String userId, required Item item}) async {
    try {
      final docRef = await _read(firebaseFirestoreProvider)
          .userListRef(userId)
          .add(item.toDocument());
      return docRef.id;
    } on FirebaseException catch (e) {
      throw CustomException(message: e.message);
    }
  }

  @override
  Future<void> updateItem({required String userId, required Item item}) async {
    try {
      await _read(firebaseFirestoreProvider)
          .userListRef(userId)
          .doc(item.id)
          .update(item.toDocument());
    } on FirebaseException catch (e) {
      throw CustomException(message: e.message);
    }
  }

  @override
  Future<void> deleteItem(
      {required String userId, required String itemId}) async {
    try {
      await _read(firebaseFirestoreProvider)
          .userListRef(userId)
          .doc(itemId)
          .delete();
    } on FirebaseException catch (e) {
      throw CustomException(message: e.message);
    }
  }
}
```

個人的にFirestoreからデータを取ってくる際、使いまわす部分はこのように

```dart:firebase_firestore_extension.dart
import 'package:cloud_firestore/cloud_firestore.dart';

extension FirebaseFirestoreX on FirebaseFirestore {
  CollectionReference userListRef(String userId) =>
      collection('lists').doc(userId).collection('userList');
}
```

extensionでまとめてしまうのが良いと思いました！

# まとめ
正直まだまだわからないことだらけですが、興味のあることや勉強になったことはなるべく記事にしてアウトプットしていこうと思います！
少しばかり長めの記事になってしまいました。今後はこれらの技術を使って実際にアプリも作っていきたいと思います！

ここまで読んでくださった方本当にありがとうございます！！



