---
title: "１ヶ月で本格的な Flutter アプリ開発（リポジトリも公開）" # 記事のタイトル
emoji: "🌟" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Firebase"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに

ご覧いただきありがとうございます。[gan](https://zenn.dev/ryota_iwamoto)です。
突然ですが最近、自分ともう1人のエンジニアで１ヶ月でFlutter製のアプリをリリースしました！
この記事ではどのような流れでアプリを開発したのかをざっくりまとめています。よければみなさんのアプリ開発の参考にしてください！

:::message alert
この記事では Flutter のインストール方法や Firebase のセットアップ、アプリのリリース方法説明しません。その点はご容赦ください。
:::

# コードはすべて公開しています！！

https://github.com/Ryotaewamoto/bad-log

本記事で紹介するアプリのコードを公開しています！是非参考にしてください！！
リリースはしていますが、まだまだ至らぬ点も多いとは思います。もちろん issue 等で指摘していただけると今後の励みになります！
よろしくお願いします！

# どんなアプリを作ったのか

今回自分が作ったのはバドミントンの試合結果を記録するためのアプリです。バドミントンの試合結果を記録？？と思う方がほとんどだと思います。このアプリを作ろうとしたきっかけは、自分の友達が試合結果を家に帰ってからパソコンで Excel に入力している、というのを聞き、これはチャンスだ！と思ったことです。2023年始まってすぐだったのですが、急いで Figma でざっくりデザインを作り、アプリを作っていきました。そのため、制作期間はだいたい2023年1月上旬~2023年2月上旬になります。

![](https://storage.googleapis.com/zenn-user-upload/dfbba3e8b848-20230304.png)

# 使用技術

メインは Flutter と Firebase になります。

Flutter の他に使用した主なパッケージは

- hooks_riverpod (状態管理)
- flutter_hooks (状態管理)
- freezed (モデルの immutable 化)

になります。

画面遷移の方法として [go_router](https://pub.dev/packages/go_router) がありますが、今回はもう1人のエンジニアが初心者だったということもあり使用していません。

もっと詳しく知りたい方は pubspec.yaml から確認してみてください。

:::message
今回のアプリでは管理画面が不要であったり、共通化したい部分もなかったので [melos](https://melos.invertase.dev/) 等は使用していません。
:::

また開発の特徴として、

- [fvm](https://fvm.app/) を使った Flutter のバージョン管理
- 開発環境と本番環境の環境分け
- GitHub Actions による CI
- dependabot の追加
- Firebase のセキュリティルール

が挙げれられます。

## fvm を使った Flutter のバージョン管理

2人とはいえチームでの開発なので Flutter のバージョンを統一できるよう、fvm を導入しました。asdf もありますが自分は fvm 派です。詳しくは以下の記事が参考になるのでおすすめです。

https://zenn.dev/altiveinc/articles/flutter-version-management

https://zenn.dev/altiveinc/articles/asdf-flutter

## 開発環境と本番環境の環境分け

環境分けに関して、 dev（開発） の prod（本番） の二つに分けました。もう少し規模の大きいアプリであったりテストするユーザがいる場合には staging（ステージング） を作成しても良いと思います。参考にしたのは以下の記事になります。ベースは Firebase CLI を使用し、アイコン等の環境わけに dart-define を使用しました。

https://zenn.dev/flutteruniv_dev/articles/20220904-151314-flutter-fire-flavor

https://zenn.dev/altiveinc/articles/separating-environments-in-flutter

## GitHub Actions による CI

CI では、コードのフォーマットと静的解析を行なっています。今後はテストも行えるようにしていきます。

個人開発でもチーム開発でも、コードの保守性を維持するためにもCIを構築しておくのはいいことだと思います。

https://github.com/Ryotaewamoto/bad-log/blob/develop/.github/workflows/flutter_main.yaml

## dependabot の追加

[Dependabot security update](https://docs.github.com/ja/code-security/dependabot/dependabot-security-updates/about-dependabot-security-updates) を使用することでパッケージの依存関係などセキュリティの強化につながります。[Dependabot version updates](https://docs.github.com/ja/code-security/dependabot/dependabot-version-updates/about-dependabot-version-updates) (パッケージのバージョンの更新)は今後導入していこうと思います。

https://zenn.dev/dzeyelid/articles/e36d439cdeda5edb7ddc

## Firebase のセキュリティルール

Firebase のセキュリティルールは疎かになりがちです。このアプリのリポジトリではセキュリティルールに関しても記載しているので参考にしてみてください。自分が参考にしているのは [Firebase のドキュメント](https://firebase.google.com/docs/rules/basics?hl=ja)と以下の記事になります！

https://qiita.com/KosukeSaigusa/items/18217958c581eac9b245

# その他勉強になった記事

https://zenn.dev/flutteruniv_dev/articles/20221214-090833-flutter-async-value

https://codewithandrea.com/articles/flutter-riverpod-async-notifier/

https://zenn.dev/tama8021/articles/0920_flutter_withdrawal

https://zenn.dev/moga/articles/appcheck_looks_good

https://zenn.dev/mamushi/articles/hide_generated_file_diff

# 最後に

今後はこのアプリのアップデートであったり、テストの記述、CD の構築、README の拡充などを行なっていく予定です。ですので、リポジトリにスター🌟つけていただけると嬉しいです。👀

最後まで読んでいただきありがとうございました！
