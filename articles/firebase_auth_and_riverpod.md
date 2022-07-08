---
title: "【中級者向け】Firebase Authを用いたRiverpod入門" # 記事のタイトル
emoji: "🐔" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Dart", "Firebase"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに
ご覧いただきありがとうございます。[gan](https://zenn.dev/ryota_iwamoto)です。
RiverpodとFirebase Authを使いたい、、、けど、あんまり情報ない、、、
そう思ってしまっいまいました。ということでRiverpodとFirebase Authのテンプレート的なコードを考えていこうと思います。
こういう実装はどうですか？という提案ですので、こうした方がいい、その書き方はあまり良くない、ということがあればコメントください！おなしゃす！！
それではよろしくお願いします！

:::message
UIやWidgetの切り出し、エラーハンドリングなどはあまり触れていないのでご容赦ください。
:::

# 対象者
- Riverpod、認知はしてる
- Riverpodについてもっと詳しく知りたい方
- 認証にFirebase Authを使いたい方

# 今回の内容
- Firebase Authのメールアドレスとパスワードを使ったサインインの実装（全体のコードはgithubから見れます！ぜひ見てね！！）
- Riverpodの使用したProviderの解説（メイン）

# 全体のコード

https://github.com/Ryotaewamoto/riverpod_firebase_auth

# 完成品（githubの方が見やすいかも）

## ログイン

![](https://storage.googleapis.com/zenn-user-upload/b1232dfbdc4e-20220709.gif)

## 新規登録→ログイン

![](https://storage.googleapis.com/zenn-user-upload/8db5cae12628-20220709.gif)

# pubspec.yamlの構成
```diff yaml
environment:
  sdk: ">=2.12.0 <3.0.0"

dependencies:
  flutter:
    sdk: flutter

  cupertino_icons: ^1.0.2
+  firebase_auth: ^3.3.16
+  hooks_riverpod: ^1.0.3
+  flutter_hooks: ^0.18.3

dev_dependencies:
  flutter_test:
    sdk: flutter

  flutter_lints: ^1.0.0

  firebase_core: ^1.10.0
```

# Firebase Authの準備
こちらに関しては良い記事がたくさんあるので各自でお願いします。一応良さげなものを貼っておきます。

https://zenn.dev/kazutxt/books/flutter_practice_introduction/viewer/firebase_authentication

# Riverpodの解説
ここでは今回使用する、``Provider``, ``StreamProvider``, ``StateNotifierProvider``の３種類を解説していきます。

![](https://storage.googleapis.com/zenn-user-upload/60a78c60b73f-20220709.png)

## Provider
まず公式ドキュメントでは、

> Provider はプロバイダの中で最もベーシックなプロバイダであり、値を同期的に生成してくれます。

というふうに書かれています。これがもっとも基本的なことです。
ただし、今回はこの**値を同期的に生成**を活用しているところはありません。２点ほど例があるのでそれぞれ見ていきます。

１つ目のProviderの値を同期的に生成という使い方ではない場所としては以下があります。

```dart:auth_repository.dart
final firebaseAuthProvider =
    Provider<FirebaseAuth>((ref) => FirebaseAuth.instance);
```

これは``FirebaseAuth.instance``を

- 外部から変更できない
- グローバルに宣言できるようにする

という点から基本的な``Provider``を使っています。このようにProviderの**値を同期的に生成**という使い方がメインにならない場合もあります。


そして２つ目は以下のコードです。

```dart:auth_repository.dart
// AuthRepository Provider を定義（ref.read を AuthRepository に渡す）
final authRepositoryProvider =
    Provider<AuthRepository>((ref) => AuthRepository(ref.read));
```

この部分において、``AuthRepository``に（変更する）値がなく、あくまで引数に``ref.read``をとっているだけなので値に対して同期的な処理を行なっていません。（正確には同期的な処理を行なっていますが、それはあまり考えなくて良いということです。）

次の疑問として、なぜ``AuthRepository``の引数が``ref.read``なのかが挙げられます。なぜref.watchにしないのかというと、``AuthRepository``クラスの中身を見るとわかります。

```dart:auth_repository.dart
class AuthRepository implements BaseAuthRepository {
  final Reader _read;

  const AuthRepository(this._read);

  @override
  Stream<User?> get authStateChanges =>
      _read(firebaseAuthProvider).authStateChanges();

  @override
  Future<void> signInWithEmail(String email, String password) async {
    try {
      await _read(firebaseAuthProvider)
          .signInWithEmailAndPassword(email: email, password: password);
    } on FirebaseAuthException catch (e) {
      throw convertAuthError(e.code);
    }
  }

  @override
  Future<UserCredential> signUp(String email, String password) async {
    try {
      final result =
          await _read(firebaseAuthProvider).createUserWithEmailAndPassword(
        email: email,
        password: password,
      );
      return result;
    } on FirebaseAuthException catch (e) {
      throw convertAuthError(e.code);
    }
  }

  @override
  User? getCurrentUser() {
    try {
      return _read(firebaseAuthProvider).currentUser;
    } on FirebaseAuthException catch (e) {
      throw convertAuthError(e.code);
    }
  }

  @override
  Future<void> signOut() async {
    try {
      await _read(firebaseAuthProvider).signOut();
    } on FirebaseAuthException catch (e) {
      throw convertAuthError(e.code);
    }
  }
}
```

それぞれ、サインイン、サインアップ、現在のユーザを取得、サインアウトを行なっています。
``Reader``を使っているというのもありますが、``_read(firebaseAuthProvider)``のように``firebaseAuthProvider``を読み取るだけ、というのが大きな理由です。Riverpodは基本的に``ref.watch``を使用していきます。なので``ref.read``の必要があるとことは出てきたら覚えておいて、それ以外は``ref.watch``を使っていくと良いと考えます。

## 補足：Reader型とは

Andrea Bizzottoさんのツイートがとてもわかりやすいので貼っておきます。各RepositoryのProviderを何個も引数に取らないので非常に便利です。

https://twitter.com/biz84/status/1534773316145356801?s=20&t=O9H9n1u8-UKpgmBQBnEYhQ


ただ注意として、Readerが使えるのはRepositoryの層がデータの受け渡しをする役割である場合に限られる点です。
``~RepositoryProvider``が``.autoDispose``付きのものだと、参照されなくなったタイミングで``~RepositoryProvider``が破棄されてしまうので要注意。


https://riverpod.dev/ja/docs/providers/provider/

## StreamProvider

こちらも公式ドキュメントの内容を引用すると、

> - Firebase や WebSocket の監視するため。
> - 一定時間ごとに別のプロバイダを更新するため。

以上のような用途で使われます。StreamBuilderがFlutterにはあるのでいつ使うの？と疑問に思います。この``StreamBuilder``が輝けるのは以下のような場合です。

```dart:user_state_provider.dart
final userStateProvider = StreamProvider((ref) {
  return ref.watch(authRepositoryProvider).authStateChanges;
});
```

```dart:auth_verification.dart
class AuthVerification extends HookConsumerWidget {
  const AuthVerification({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final authState = ref.watch(userStateProvider);
    return authState.when(data: (data) {
      // ログイン済みの場合
      if (data != null) {
        return const MyHomePage();
      }
      // 未ログインの場合
      return const SignInPage();
    }, loading: () {
      return const Scaffold(
        body: Center(
          child: CircularProgressIndicator(),
        ),
      );
    }, error: (_, __) {
      return const Scaffold(
        body: Center(
          child: Text('エラーだよ'),
        ),
      );
    });
  }
}
```

このようにAsyncValueによってloading/errorのステートを適切に処理することができるのはとても良いです。
ただ個人的な意見ですが、StreamProviderやFutureProviderを適切なタイミングで使うのは一朝一夕でできるものではなく、かなり玄人向けかなと思います。
ですが、これが使えるとAsyncValueの点からもコードの保守性がかなり高くなると思うのでチャレンジはしてみたいです。


## StateNotifierProvider

まず、公式ドキュメント。

> - 「イミュータブル（不変）」 なステートを公開するため（イミュータブルではあるが、イベントに応じて変わることがある）。
> - ステートを変更するためのロジック（いわゆるビジネスロジック）を一つの場所で集中管理して保守性を高めるため。

と記述してあります。今回のようにただの認証だけであれば``StateNotifierProvider``を使わないで行ける気がしますが、WidgetからRepositoryProviderを直接呼ぶのを避けるために自分は以下のように作成しました。

```dart:auth_provider.dart
final authControllerProvider = StateNotifierProvider.autoDispose<AuthController, User?>(
  (ref) => AuthController(ref.read),
);

class AuthController extends StateNotifier<User?> {
  final Reader _read;

  AuthController(this._read) : super(null);

  @override
  User? get state => _read(authRepositoryProvider).getCurrentUser();

  @override
  void dispose() {
    super.dispose();
  }

  Future<void> signIn(String email, String password) async {
    try {
      await _read(authRepositoryProvider).signInWithEmail(email, password);
    } catch (e) {
      throw e.toString();
    }
  }

  Future<void> signUp(String email, String password) async {
    await _read(authRepositoryProvider).signUp(email, password);
    // Firestoreにユーザデータを追加したり
  }

  Future<void> signOut() async {
    await _read(authRepositoryProvider).signOut();
  }
}
```

強いて言うのであれば

```dart:auth_provider.dart
  @override
  User? get state => _read(authRepositoryProvider).getCurrentUser();
```

は、ユーザをステートとして持たせるアプリ（ほぼそう）であればFirestore等のデータベースからUserクラスを作って、それを``StateNotifierProvider``を使ってイミュータブルに操作していく、と言うのが定石なのかなと思います。

## 補足:.autoDispose

```dart:auth_provider.dart
final authControllerProvider = StateNotifierProvider.autoDispose<AuthController, User?>(
  (ref) => AuthController(ref.read),
);
```

最後に少しだけ補足させてください。(gif作ってる時に気づいた)

ここの部分に関して、``.autoDispose``がないとどうなるか説明します。
test01@gmail.comというメールアドレスでログイン後、ログアウトします。その後test02@gmail.comという別のメールアドレスでログインすると画面には「test01@gmail.com」という前にログインしたアカウントのメールアドレスが表示されてしまいます。
これは何が起こっているかというとtest01@gmail.comでログインした際にできた``StateNotifierProvider``のステートがログアウト後も残っており、このコードでは特に何もステートを変更していないので前のメールアドレスが表示されてしまうというわけです。ステートの管理には気をつけたいです。


# まとめ

いかがだったでしょうか。Riverpodと少しでも仲良くなれたと思っていただけたら嬉しいです。

自分も正直まだわからない点が山ほどあるので少しずつ記事として残していこうと思います！

最後まで読んでいただきありがとうございました。さらばっ！

# 参考文献
- https://riverpod.dev/ja/docs/concepts/providers/
- https://riverpod.dev/ja/docs/concepts/modifiers/auto_dispose/
- https://note.com/mxiskw/n/n5c06bc2dd0d5
- https://zenn.dev/riscait/books/flutter-riverpod-practical-introduction/viewer/migrate-to-v1