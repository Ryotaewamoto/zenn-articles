---
title: "【Flutter】Firebase Auth による認証の実装から学ぶ Riverpod v2" # 記事のタイトル
emoji: "🐔" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Dart", "Firebase"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

:::message
2023-03-24 更新:

Riverpod v2 に対応した記事に変更しました。
:::

# はじめに
ご覧いただきありがとうございます。[gan](https://zenn.dev/ryota_iwamoto)です。
Riverpod と Firebase Auth を使いたい、、、けど、あんまり情報ない、、、
そう思ってしまっいまいました。ということで Riverpod と Firebase Auth のテンプレート的なコードを考えていこうと思います。
こういう実装はどうですか？という提案ですので、こうした方がいい、その書き方はあまり良くない、ということがあればコメントください！おなしゃす！！
それではよろしくお願いします！


# 対象者
- Riverpod、認知はしてる
- Riverpodについてもっと詳しく知りたい方
- 認証にFirebase Authを使いたい方

# 今回の内容
- Firebase Authのメールアドレスとパスワードを使ったサインインとサインアップ、サインアウトの実装（全体のコードはgithubから見れます！ぜひ見てね！！）
- Riverpod のプロバイダ(Provider, StreamProvider, NotifierProvider, AsyncNotifierProvider)の解説

# 全体のコード

以下のリポジトリに動画も添付しているので参考にしてみてください！

https://github.com/Ryotaewamoto/riverpod_firebase_auth

# フォルダ構成

![](https://storage.googleapis.com/zenn-user-upload/24562870b10f-20230328.png)

# pubspec.yamlの構成
```diff yaml
environment:
  sdk: ">=2.19.0 <3.0.0"

dependencies:
  flutter:
    sdk: flutter

  cupertino_icons: ^1.0.2
+  firebase_auth: ^4.3.0
+  firebase_core: ^2.8.0
+  hooks_riverpod: ^2.3.2

dev_dependencies:
  flutter_test:
    sdk: flutter

  flutter_lints: ^1.0.0

```

# Firebase Authの準備
こちらに関しては良い記事がたくさんあるので各自でお願いします。一応良さげなものを貼っておきます。

https://zenn.dev/kazutxt/books/flutter_practice_introduction/viewer/firebase_authentication

# Riverpodの解説
ここでは今回使用する、``Provider``, ``StreamProvider``, ``NotifierProvider``, ``AsyncNotifierProvider``の4種類を解説していきます。

:::message

2023-03-24 更新内容:

もともと使用していた ``StateNotifierProvider`` の項目を削除し、``NotifierProvider``, ``AsyncNotifierProvider`` の2つを追加しました。
:::

## 前提
まず公式ドキュメントでは、プロバイダーが必要な理由として

> ステートをプロバイダでラップすることで次のことが可能になります。
>
> (1)アプリの様々な場所からステートにアクセスできるようになります。 つまり、プロバイダはシングルトンやサービスロケータのようなパターン、依存性注入、あるいは InheritedWidget を完全に代替することができます。
>
> (2)ステートを別のプロバイダのステートと簡単に組み合わせることができるようになります。 開発では複数のオブジェクトを組み合わせて一つのステートにまとめるのに四苦八苦する場面も多いかと思います。プロバイダにはこのための機能が組み込まれています。
>
> (3)アプリのパフォーマンスを最適化してくれます。 例えば、ウィジェット更新の条件を限定したり、負荷が高いステートの計算をキャッシュしたりといったことが可能になります。 プロバイダがステートの変化による外部への影響をコントロールしてくれます。
>
> (4)アプリのテスト容易性を高めてくれます。 プロバイダがあれば setUp や tearDown のような面倒な手順は不要です。 さらに、テスト中のプロバイダの挙動をオーバーライドすることができます。 これにより特異な条件下での動作も確認しやすくなります。
>
> (5)ロギングやプルリフレッシュ（画面を引っ張って更新）などの高度な機能との組み合わせが容易に実現できます。

と書かれています。このことを前提にそれぞれの Provider の用途について説明してきます。

## Provider

まずは基本的な ``Provider`` です。さっそく使用例を見ていきましょう。

- 使用例１:

```dart:lib/pages/sigh_up_page.dart
final _nameTextEditingController =
    Provider.autoDispose<TextEditingController>(
  (_) => TextEditingController(),
);
```

主な使用理由は前提の(1)になります。こうすることで、サインアップ画面から ``TextEditingController`` のインスタンスにアクセスできるようにしています。

また、参照されなくなった際に ``TextEditingController`` は破棄されて欲しいので　``.autoDispose`` をつけています。

- 使用例２:

```dart:lib/repositories/auth_repository.dart
final firebaseAuthProvider =
    Provider<FirebaseAuth>((ref) => FirebaseAuth.instance);
```

こちらは同ファイル内で ``AuthRepositoryImpl`` クラスの引数として取るために Provider で定義しています。

```dart
final _instance = FirebaseAuth.instance;
```

としても動作します。ものすごい恩恵があるわけではないですが、Provider にしておくことの良い点は ``ref.watch`` でのみと参照できるのでファイル内の任意の場所から呼び出すことを避けることができます。

- 使用例3:

```dart:lib/auth_repository.dart
final authRepositoryImplProvider = Provider<AuthRepository>(
  (ref) => AuthRepositoryImpl(ref.watch(authProvider)),
);
```

ここでの使用目的は上の前提の(1)、(2)に当たります。使用例2の ``Provider`` を組み込んでいるのがわかると思います。

:::message alert
2023-03-24 更新内容:
更新前の記事では ``Reader`` というクラスを用いた書き方をしていましたが、Riverpod のバージョンが2以降の場合にはこの ``Reader`` は使えないのでこの記事からは削除しました。
:::

補足として ``AuthRepositoryImpl`` は以下のようにしています。Firebase Auth を使用する処理はここにまとめています。

:::details lib/repositories/auth_repository_impl.dart
```dart
class AuthRepositoryImpl implements AuthRepository {
  AuthRepositoryImpl(this._auth);
  final FirebaseAuth _auth;

  @override
  User? get currentUser => _auth.currentUser;

  @override
  Stream<User?> authStateChanges() => _auth.authStateChanges();

  @override
  Future<String?> signUp({
    required String email,
    required String password,
  }) async {
    final userCredential = await _auth.createUserWithEmailAndPassword(
      email: email,
      password: password,
    );

    return userCredential.user?.uid;
  }

  @override
  Future<void> signIn({
    required String email,
    required String password,
  }) async {
    await _auth.signInWithEmailAndPassword(
      email: email,
      password: password,
    );
  }

  @override
  Future<void> sendPasswordResetEmail({required String email}) async {
    await _auth.sendPasswordResetEmail(email: email);
  }

  @override
  Future<void> signOut() async {
    await _auth.signOut();
  }
}
```
:::

## StreamProvider

こちらも公式ドキュメントの内容を引用すると、

> - Firebase や WebSocket の監視するため。
> - 一定時間ごとに別のプロバイダを更新するため。

以上のような用途で使われます。``StreamBuilder`` が Flutter に標準で用意されているため、いつ使うの？と疑問に思います。この ``StreamProvider`` が輝けるのは以下のような場合です。

```dart:auth_repository_impl.dart
final userStateProvider = StreamProvider((ref) {
  return ref.watch(authRepositoryProvider).authStateChanges;
});
```

```dart:auth_page.dart
class AuthVerification extends HookConsumerWidget {
  const AuthVerification({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    ref.handleConnectivity();

    return Scaffold(
      body: ref.watch(authUserProvider).when(
        data: (data) {
          if (data != null) {
            // data が null でない、つまりログイン済みの場合はホーム画面へ
            return const HomePage();
          } else {
            // data が null のとき、つまり未ログインの場合はログイン画面へ
            return const GetStartedPage();
          }
        },
        error: (error, stackTrace) {
          return const ErrorPage();
        },
        loading: () {
          return const OverlayLoadingWidget();
        },
      ),
    );
  }
}
```

``ref.watch(authUserProvider)`` の型は ``AsyncValue`` になっており、この ``AsyncValue`` を使うことによって、loading/error のステートを適切に処理することができるのでとても良いです。

## (Async)NotifierProvider

``(Async)NotifierProvider`` は ``(Async)Notifier`` クラスを継承したクラスに対して状態の監視を行うためのプロバイダです。

まず、公式ドキュメント。

> - あるに任意のイベントが発生した後に変更があるステートを公開するため
> - ステートを変更するためのロジック（いわゆるビジネスロジック）を一つの場所で集中管理して保守性を高めるため。

と記述してあります。今回のようにただの認証だけであれば``(Async)NotifierProvider``を使わないで行ける気がしますが、``Widget`` から ``AuthRepositoryImplProvider`` を直接呼ぶのを避けるために自分は以下のように作成しました。

まず、``NotifierProvider`` の使用例は以下になります。

```dart:lib/pages/login_page.dart
final _isObscureProvider =
    NotifierProvider.autoDispose<IsObscureNotifier, bool>(
  IsObscureNotifier.new,
);
```

```dart:lib/features/notifier/is_obscure_notifier.dart
class IsObscureNotifier extends AutoDisposeNotifier<bool> {
  @override
  bool build() => true;

  bool toUnobscured() => state = false;

  bool toObscured() => state = true;
}
```

``build()`` メソッドで初期化を行い、ステートの変更を行うような処理を関数で記述していきます。

この例ではパスワードを見える状態にするか、見えない状態にするかのステートを持しています。

次に、``AsyncNotifierProvider`` の使用例は以下になります。

```dart:lib/features/auth/sign_in_controller.dart
/// Firebase Auth を用いてサインインをする [AsyncNotifierProvider]。
final signInControllerProvider =
    AutoDisposeAsyncNotifierProvider<SignInController, void>(
  SignInController.new,
);

class SignInController extends AutoDisposeAsyncNotifier<void> {
  @override
  FutureOr<void> build() {
    // FutureOr<void> より、初期の処理の必要がないため何もしない。
    // Do nothing since the return type is void.
  }

  /// サインインする
  Future<void> signIn({
    required String email,
    required String password,
  }) async {
    final authRepository = ref.read(authRepositoryImplProvider);
    // サインイン結果をローディング中にする
    state = const AsyncLoading();

    // サインイン処理を実行する
    state = await AsyncValue.guard(() async {
      try {
        final isNetworkCheck = await isNetworkConnected();
        if (!isNetworkCheck) {
          const exception = AppException(
            message: 'Maybe your network is disconnected. Please check yours.',
          );
          throw exception;
        }

        if (email.isEmpty || password.isEmpty) {
          const exception = AppException(
            message: 'Please input your email and password.',
          );
          throw exception;
        }
        await authRepository.signIn(
          email: email,
          password: password,
        );
      } on FirebaseAuthException catch (e) {
        final exception = AppException(
          code: e.code,
          message: e.toJapanese,
        );
        debugPrint(e.code);
        throw exception;
      }
    });
  }
}
```

``Notifier`` と同様に ``AsyncNotifier`` でも ``build()`` メソッドで初期化を行い、ステートの変更を行うような処理を関数で記述していきます。今回のように処理だけの場合は ``void`` を持たせるようにしましょう。

この例ではサインイン時の処理をここでまとめています。

``NotifierProvider`` との違いは名前からもわかるように非同期(async)であるかどうかです。外部との通信を行うような場合はこちらを使うようにしましょう。

:::message

``AsyncValue.guard`` 自体に ``try ~ catch`` の処理があるので不要に思えるかもしれません。ただ、エラーハンドリングを細かく行いたい場合には、上のように ``AsyncValue.guard()`` の中身に ``try ~ catch`` の処理を書いて対応するようにしましょう。
:::

## 補足(flutter_hooks):

[flutter_hooks](https://pub.dev/packages/flutter_hooks) を使った場合には今回定義した ``Provider`` や ``NotifierProvider`` はもう少し簡単に書けます。

### pubspec.yamlの更新

```diff yaml
environment:
  sdk: ">=2.19.0 <3.0.0"

dependencies:
  flutter:
    sdk: flutter

  cupertino_icons: ^1.0.2
  firebase_auth: ^4.3.0
  firebase_core: ^2.8.0
  hooks_riverpod: ^2.3.2
+  flutter_hooks: ^0.18.5+1

dev_dependencies:
  flutter_test:
    sdk: flutter

  flutter_lints: ^1.0.0
```

以下に変更点を記しておきます。基本的に使いたい ``build`` メソッド内で定義するだけです。

### 変更例 1 (Provider)

```diff dart
- final _nameEmailTextEditingController =
-     Provider.autoDispose<TextEditingController>(
-   (_) => TextEditingController(),
- );
```

```diff dart
  @override
  Widget build(BuildContext context, WidgetRef ref) {
+    final userNameController = useTextEditingController();
```

### 変更例 2 (NotifierProvider)

```diff dart
- final _isObscureProvider =
-     NotifierProvider.autoDispose<IsObscureNotifier, bool>(
-   IsObscureNotifier.new,
- );

- class IsObscureNotifier extends AutoDisposeNotifier<bool> {
-   @override
-   bool build() => true;
-
-   bool toUnobscured() => state = false;

-   bool toObscured() => state = true;
- }
```

```diff dart
  @override
  Widget build(BuildContext context, WidgetRef ref) {
+    final isObscure = useState(true);
```

より詳しい使い方は以下のリポジトリを参考にしてみてください。

https://github.com/Ryotaewamoto/bad-log

# まとめ

いかがだったでしょうか。Riverpod と少しでも仲良くなれたと思っていただけたら嬉しいです。

自分も正直まだわからない点が山ほどあるので少しずつ記事として残していこうと思います！

最後まで読んでいただきありがとうございました。

# 参考文献
- https://riverpod.dev/ja/docs/concepts/providers/
- https://riverpod.dev/ja/docs/concepts/modifiers/auto_dispose/
- https://note.com/mxiskw/n/n5c06bc2dd0d5
- https://zenn.dev/riscait/books/flutter-riverpod-practical-introduction/viewer/migrate-to-v1
