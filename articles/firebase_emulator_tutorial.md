---
title: "【FlutterFire】Firebase Emulatorでローカル環境で開発しよう" # 記事のタイトル
emoji: "😲" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Firebase"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# はじめに
ご覧いただきありがとうございます。ganです。

今回はFlutter×Firebaseの開発をどこでもやってやろうじゃないか！ということで、Firebase Emulatorを使ったローカル開発について書きました。特に使用頻度の高いFirestoreとFunctionsも使います。
ほかにも開発スピードを上げられるような工夫も考えました。

まだまだわからないことだらけですが、誰かの役に立ったら嬉しいです！

またgithubにサンプルコードも上げているのでぜひ見てみてください！！

https://github.com/Ryotaewamoto/firebase_emulator_tutorial

# この記事の内容
- Firebase Emulatorとは何か
- Firebaseの導入
- Cloud FunctionsとEmulator
- Firebase AuthとEmulator
- Firebase FirestoreとEmulator

# 開発環境
- マシン: MacBook Intel (2020)
- エディタ: VSCode
- Flutter: 2.5.3 • channel stable
- Firebase: 10.0.0

## Firebase Emulatorとは何か
まず、正式名称は、Firebase Local Emulator Suiteと言います。以降、長いのでFirebase Emulatorとします。具体的に何ができるかというと、local環境でFirebase（Auth(認証)、Firestore、Realtime Database、Storage、Functions、Hosting、Pub/Sub）を使うことができます。そのためSecurity rulesやFunctionsをデプロイする前に試せるので、アプリの本稼働やプロトタイピングにかかる時間を短縮できます。



## Firebaseの導入

はじめにFirebaseのプロジェクトを作成してください

https://firebase.google.com/?hl=ja

その後はFlutterのアプリのディレクトリで``firebase init``コマンドを打ってぽちぽちしていきます。

```
firebase init #Firebaseをインストール
```
を入力してぽちぽちしていきます。


``firebase-tools``が入っていない場合は以下のコマンドをターミナルで実行したください。

```
npm install -g firebase-tools
```

インストール後は

```
firebase --version
```

でFirebaseのバージョンも確認できます。





# まとめ