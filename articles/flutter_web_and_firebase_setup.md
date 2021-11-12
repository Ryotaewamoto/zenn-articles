---
title: "[Flutter Web] [Firebase] [2021/10/09時点] Flutter Web に Firebase を追加" # 記事のタイトル
emoji: "👶" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Web", "Firebase"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに
ご覧いただきありがとうございます。ganです。

# 記事の経緯
Flutter Web とFirebaseの追加について書いていきます。

というのも、Firebaseのバージョンが8から9に上がったことで、
Flutter Web とFirebase の接続がうまくいかないとわかったからです。

なので今回は今からFlutter Web x Firebase でアプリを作っていこうとしている人むけに書いていきます。

最新の情報が出たり、問題のできるようになった場合にはコメントしていただけると嬉しいです。

# 今回の内容

## step.1 Firebaseのプロジェクトを作成

まずはFirebaseのプロジェクトを指示に従って作成してください。

Firebase公式: https://firebase.google.com/?hl=ja

## step.2 Flutterのプロジェクトを作成

Flutterのプロジェクトを作成したら、まずはWebに対応するようにします。

ローカルのターミナルで

``
 flutter create .
``

を実行してください。

そうすると、名前が「web」となっているファイルが作成されます。

中にindex.htmlが入っているのでそれを使います。

## step.3 Firebaseのプロフェクトにwebを追加

次にFirebaseプロジェクトにwebを追加します。

このとき以下の画面まで行ったら、scriptタグ内のconst firebaseConfig...のとこをコピーしておいてください。

[![Image from Gyazo](https://i.gyazo.com/47ee5d35b3075d340c70c87f7a53bd66.png)](https://gyazo.com/47ee5d35b3075d340c70c87f7a53bd66)


ここ↓
```javaScript
const firebaseConfig = {
  <!-- apiKey, messagingSEndeerId, appId はプロジェクトごとに異なるので注意 -->
    apiKey: "AIzaSyDmOFOAYr98u-PEd35sd773p6o4w83v2kw",
    authDomain: "flutter-web-firebase-setup.firebaseapp.com",
    projectId: "flutter-web-firebase-setup",
    storageBucket: "flutter-web-firebase-setup.appspot.com",
    messagingSenderId: "790144669464",
    appId: "1:790144669464:web:b23c35e81ff0780678b06d"
  };
```

## step.4 index.htmlを編集する

最後にindex.htmlのbodyタグの下に以下のコードをコピーして貼り付けてください。

その後先ほどFirebaseのプロジェクトを作成した際にコピーしたものをコードの下部のスペースに貼り付けてください。

その際に、先頭のconst をvar に変更しておいてください。

```javaScript
<!-- Google認証 -->
<script src="https://apis.google.com/js/platform.js" async defer></script>

<!-- The core Firebase JS SDK is always required and must be listed first -->
<script src="/__/firebase/8.6.1/firebase-app.js"></script>

<!-- TODO: Add SDKs for Firebase products that you want to use
  https://firebase.google.com/docs/web/setup#available-libraries -->
<script src="/__/firebase/8.6.1/firebase-analytics.js"></script>

<!-- Initialize Firebase -->
<script src="/__/firebase/init.js"></script>

<!-- The core Firebase JS SDK is always required and must be listed first -->
<script src="https://www.gstatic.com/firebasejs/8.6.1/firebase-app.js"></script>

<!-- TODO: Add SDKs for Firebase products that you want to use
  https://firebase.google.com/docs/web/setup#available-libraries -->
<script src="https://www.gstatic.com/firebasejs/8.6.1/firebase-analytics.js"></script>
<script src="https://www.gstatic.com/firebasejs/8.6.1/firebase-firestore.js"></script>
<script src="https://www.gstatic.com/firebasejs/8.6.1/firebase-auth.js"></script>
<script>
      // Your web app's Firebase configuration
      // For Firebase JS SDK v7.20.0 and later, measurementId is optional


<!-- TODO: この部分に先ほどコピーしたものを貼り付ける -->





<!----------------------------------------------->
       
      // Initialize Firebase
      firebase.initializeApp(firebaseConfig);
      firebase.analytics();
    </script>
```


## step.5(最後) yamlファイルを編集

最後にpubspec.yamlに必要なものを追加してdebugしてみてください。


(とりあえず、以下の二つを追加するのをお勧めします)
pub.dev: https://pub.dev/packages/firebase_core
pub.dev: https://pub.dev/packages/cloud_firestore

# まとめ

今回はFlutter Web にFirebase を追加するのをやっていきました。

今はFirebase9とFlutterの記事は少ないですがこれから増えて行くと思います。

そしたこの記事はあまり必要なくなってしまいますが、その分Flutterが盛り上がっているということのなのでととても嬉しく思います！

最後まで読んでいただきありがとうございます！！

# 参考

https://firebase.google.com/docs/web/modular-upgrade
https://flutter.dev/docs/get-started/web

github: https://github.com/Ryotaewamoto/flutter_web_firebase_setup
