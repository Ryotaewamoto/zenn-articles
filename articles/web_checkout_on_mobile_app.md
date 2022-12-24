---
title: "モバイルアプリでWeb上決済を導入する" # 記事のタイトル
emoji: "👶" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "firebase", "stripe", "ios", "android"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

:::message
こちらは Flutter 大学 Advent Calendar 2022 25 日目の記事です!
:::

https://qiita.com/advent-calendar/2022/flutteruniv

# キーワード
- Flutter(もしくは、iOS, Android, Web)
- Firebase Dynamic Links
- Cloud Functions
- Stripe checkout
- サブスクリプション(定期決済)
- Stripe と Firestore との連携

# 対象の読者
- 決済には興味があるがあまり実装のイメージがつかないと感じている方
- 実際に、実装のコストを抑えて決済機能を導入したい方

# はじめに
ご覧いただきありがとうございます。[gan](https://zenn.dev/ryota_iwamoto)です。

この記事を通して、モバイルアプリに”Web上で行う”決済の導入方法を知っていただけたらと思います。実際に導入することも意識して書きました。その部分も注目していただけると幸いです。またクライアント側にはFlutterを使用しますが、iOS、Android、加えてWebのフロントエンドでもほとんど同じように実装できます。（Firebase の設定やクライアント側のコードは異なりますが）。
少し長い記事になりますので時間をとってじっくり見ていただけたら嬉しいです。
また、至らぬ点があればコメントしてもらえると私も勉強になります。
ぜひよろしくお願いします！

# 実装内容（仕様）
実装内容をイメージできるように簡単な仕様について先に書いておきます。あくまで決済部分に焦点を当てるため、Firebase の設定やStripeの準備等は割愛します。

## 実装したいこと（重要）
- アプリ内にユーザが存在し、ユーザはサブスクリプションを購読することができる。
- アプリ内のユーザは、アプリ内部からサブスクリプションを解除することができる。
- 存在するサブスクリプションのプランは一つとする。

## 実装イメージ
決済部分のイメージ図になります。

![](https://storage.googleapis.com/zenn-user-upload/8a56ca0048ce-20221224.png)

## 技術選定
あくまでざっくりとした内容です。

- ユーザのカード情報はこちらで管理したくないので **Stripe** を使用します。
- クライアントは **Flutter** を使用します。
- データベースは **Firestore** を使用します。
- サーバーサイドは **Cloud Functions** で対応します。

## 事前知識
具体的なデータベース設計や実装に入る前にこの実装の要となる**Stripe Checkout**と**Firebase Dynamic Links**について説明します。

### Stripe Checkout
Stripe Checkout とは、

> Checkout のローコードの組み込みを使用し、[1 回限りの支払い](https://stripe.com/docs/payments/accept-a-payment)や[サブスクリプション](https://stripe.com/docs/billing)を受け付ける Stripe がオンラインで提供する決済ページを追加します。Checkout の事前構築された機能により、開発時間が短縮されます。[Checkout Session API](https://stripe.com/docs/api/checkout/sessions) と Stripe ダッシュボードを使用して追加機能にアクセスし、ビジネスに合わせて Checkout をカスタマイズできます。機能の一覧については、[構築済みの機能とカスタマイズ可能な機能](https://stripe.com/docs/payments/checkout/how-checkout-works#features)をご確認ください。

（引用元: https://stripe.com/docs/payments/checkout/how-checkout-works?locale=ja-JP）

と、公式の方で説明されています。書いてある通りですでにわかりやすいのですが、もっとわかりやすく『関西弁のおばちゃん風に』言うのであれば、『**面倒な決済の部分はうち（Stripe）が用意した決済画面があるからそれを使いー。カスタマイズもそこそこできるでー**』という感じです。次に実装の部分を説明します。以下の図を見てください。

![](https://storage.googleapis.com/zenn-user-upload/8def359230d8-20221215.png)

こちらは Stripe が公式で出している図になります。こちらの図も非常にわかりやすいのでこちらで説明することはないのですが、上の実装イメージの図と照らし合わせてみるとより図をイメージしやすいと思うのでそちらを推奨します。

Web の場合はそのままリンク先に遷移させればいいのでが、モバイルアプリの場合はそううまくはいきません。そこで登場するのが後述する Firebase Dynamic Links です。

### Firebase Dynamic Links
Firebase Dynamic Links とは、

> Firebase Dynamic Linksは、複数のプラットフォームで、アプリが既にインストールされているかどうかに関係なく、希望どおりに機能するリンクです。ダイナミックリンクを使用すると、ユーザーはリンクを開いたプラットフォームで利用可能な最高のエクスペリエンスを得ることができます。ユーザーがiOSまたはAndroidでダイナミックリンクを開くと、ネイティブアプリのリンクされたコンテンツに直接移動できます。ユーザーがデスクトップブラウザで同じダイナミックリンクを開くと、Webサイトの同等のコンテンツに移動できます。

ということです。自分が説明をする必要がないぐらいわかりやすいのですが、改めて『関西弁のおばちゃんを召喚して』説明すると、『**うち（Firebase）の Dynamic Links はすごいでー。この　Dynamic Links を使えば、Webサイトからアプリの画面に（インストールに関わらず）飛ばすことができるんやでー**』という感じです。

今回の実装で Dynamic Links がどこで活躍するのかというと、**決済完了後にアプリの画面に戻す**ときになります。

## データベース設計
最低限のフィールドのみ記述します。そのため実際に導入する際は必要に応じてフィールドを追加してください。

また、Firestore は NoSQL です。SQL を使う場合にはデータベース設計は異なると思いますので、その点はご注意ください。

```json
{
    "collection: appUsers": { // ユーザ
        "document: {appUserId}": {
            "appUserId": "{appUserId}",
            "customerId": "cus_xxxxxxxx",
            "createdAt": "2022-12-25 15:00",
            "collection: subscriptions": { // サブスクリプション
                "document: {subscriptionId}": {
                    "subscriptionId": "{subscriptionId}",
                    "stripeSubscriptionId": "sub_xxxxxxxx",
                    "status": "active",
                    "createdAt": "2022-12-25 15:00"
                }
            },
        },
    },
}
```

## 主要なフィールドの解説
以下、Stripe周りで必要になるフィールドがあるので解説します。

### customerId: 顧客情報
customerId とは、Stripe上のユーザ情報を含むエンティティ（Customer（＝顧客））のIDになります。ここにはユーザのメールアドレスやクレジットカード情報が含まれます。定期決済が何かしらの理由により失敗した場合にはcustomerに含まれるメールアドレスに支払いを請求することができます。より詳細な情報は[こちら（Stripeのドキュメント）](https://stripe.com/docs/billing/customer?locale=ja-JP)を参照してください。
こちらはユーザを作成する、つまりはサインアップの際に customerId を作成します。作成する関数は Firebase Cloud Functions を使用します。

### stripeSubscriptionId: サブスクリプション情報
stripeSubscriptionId は、Stripe 上のサブスクリプションの情報を特定するために使用するフィールドです。Stripe では Subscription というオブジェクトの　``id``　で取得できます。subscriptionId というフィールド名でも良いのですが、アプリのデータベース上の名前（Firestore のフィールド名）と区別するために　``stripe``　という接頭辞を追加しています。より詳細な Subscription オブジェクトの情報は[こちら（Stripeのドキュメント）](https://stripe.com/docs/api/subscriptions/object)を参照してください。
こちらは具体的なサブスクリプションの登録をする際に作成します。作成は、 customerId と同様に Firebase Cloud Functions を使用して行います。

### status: サブスクリプションの状態
status は、作成したサブスクリプションの状態を表す値になります。Stripe では status の値として、``incomplete``, ``incomplete_expired``, ``trialing``, ``active``, ``past_due``, ``canceled``, または ``unpaid`` を取ることができます。ただし今回使用するのは基本的に``incomplete``, ``active``, ``canceled``しか使いません。より詳細な status の情報は[こちら（Stripeのドキュメント）](https://stripe.com/docs/api/subscriptions/object)を参照してください。
こちらもFirebase Cloud Functions を使用して変更を行います。Stripe では Webhook (何かしらのオブジェクトの作成や更新に対して、それが行われたタイミングで何か登録した処理を行うこと)が用意されているのでこれを使用して Firestore 上の status の値を変更します。詳しくは以下の　Stripe の部分で解説します。

# 実装手順

1. 必要なものを準備
2. Stripe（商品の登録）
3. Firebase Dynamic Links
4. Cloud Functions（サーバーサイド)
5. Stripe（Webhookの設定）
6. Flutter(クライアント)

## 必要な準備
以下に関しては事前に用意しておく必要があります。使用の用途の説明も加えておきました。

- ユーザがログインする画面（Flutter）
理由: ユーザに customerId を付与

- アプリの App store の登録
理由: iOS の Dynamic Links の登録時に App Store ID が必要

- Firebase 初期化
理由: Firestore, Cloud Functions, Dynamic Links の使用

- Stripe のアカウントの作成
理由: 決済機能の使用

- Stripe のサブスク商品
理由：決済リンクを作成する際に必要

# 実装
ここからは具体的なコードを用いて実装の解説をしていきます。あくまで一例なのでメインの処理を残しつつカスタマイズして使っていただけると幸いです。

## Stripe（商品の登録）
こちらはブラウザでぽちぽちしていくだけの作業です。登録したい商品の情報を入力していってください。最終的に使うのは``price_xxxxxxxx``の部分です。

Stripe を開き、『商品』のところから商品の情報を追加します。

![](https://storage.googleapis.com/zenn-user-upload/773d8baea54a-20221221.png)

入力後の画面は以下になります。

![](https://storage.googleapis.com/zenn-user-upload/bb63e6a844a9-20221221.png)

## Firebase Dynamic Links
まずは Firebase Dynamic Links の設定から行っていきます。最終的にはこのようになればOKです。URLの末尾のアルファベットは違っていて大丈夫です。

画像のように決済成功用リンクと決済キャンセル用のリンクを用意してください。

![](https://storage.googleapis.com/zenn-user-upload/cb09a43cf6dd-20221221.png)

以下の手順に沿って進めてください。

1. Firebase のコンソールを開き、Dynamic Links を有効にする

![](https://storage.googleapis.com/zenn-user-upload/5fc0a5fe06ca-20221221.png)

2. Dynamic Links として使用するドメインを設定する
『はじめる』のボタンを押して以下のような画面で使用するドメインを登録してください。

![](https://storage.googleapis.com/zenn-user-upload/6f856dab675d-20221221.png)

3. 新しいダイナミックリンクを作成する
『新しいダイナミックリンクを作成』のボタンを押します。すると以下の画面に進むので手順に沿って入力してください。

![](https://storage.googleapis.com/zenn-user-upload/cf963f701490-20221221.png)

ダイナミック リンクの設定で入力するリンクは有効なもの（例えばアプリのLPのリンクや自分のブログのリンクなど）にしてください。この時、**成功時のURLの末尾には``?checkout_result=success``、キャンセル時のURLの末尾には``?checkout_result=cancel``を追加してください**。これによってブラウザ上での決済に足して、アプリ（クライアント）側でそれが成功したかどうかを判定します。

入力が終わったら次に進み、iOS と Android のアプリを開くようするためにそれぞれ Firebase に登録しているアプリを選択してください。

![](https://storage.googleapis.com/zenn-user-upload/f9962d6d23e6-20221221.png)

![](https://storage.googleapis.com/zenn-user-upload/77f5abc5e97b-20221221.png)

:::message alert
ここで iOS の場合、Firebase の設定で iOS アプリの App Store ID を登録している必要があります。
:::

参考:
https://firebase.google.com/docs/dynamic-links

## Cloud Functions（サーバーサイド)
まずは Cloud Functions の方で必要になる関数を定義しておきます。具体的に以下の関数が必要になります。
（たぶんここの Cloud Functions で関数を用意するところが一番大変。頑張っていきましょう！）

- ユーザに customerId を付与するための関数
- Stripe Checkout の決済リンクを作成する関数
- 決済完了時に Firestore 上に Subscription ドキュメントを作成する関数
- Stripe の Subscription の status を Firestore に反映する関数
- Stripe の Subscription を解除する関数

関数作成後は、忘れずに cloud functions のデプロイを行なってください。

### ユーザに customerId を付与するための関数
こちらは Stripe API を使用します。あらかじめシークレットキー等の設定を済ませておいてください。コードは以下のようになります。

こちらはユーザのメールアドレスを引数として、Stripe 上にメールアドレスの紐付いた customer を作成する関数になります。この後クライアント側でこの関数を呼ぶため``onCall``を使用し、Firestore に値を追加するために返り値を customerId にしています。（別の実装としては、Firestore でユーザドキュメントを作成後に``onCreate``でこちらの関数を呼び、ユーザのフィールドに customerId を追加する方法もあります。）

```ts
// Stripe の customer を作って customerId を返す関数
export const createCustomer = functions
  .region('asia-northeast1')
  .https.onCall(async (data, context) => {
    const email = data.email
    const customer = await stripe.customers.create(
      { email: email },
      { idempotencyKey: data.idempotencyKey }
    )
    const customerId = customer.id
    return { customerId: customerId }
  })
```

参考:
https://stripe.com/docs/api/customers/create

### Stripe Checkout の決済リンクを作成する関数
こちらも上と同様に Stripe API を使用します。コードは以下のようになります。

ここで重要なのが、Stripe Checkout では customerId を引数に取れることです。Stripe PaymentLinks ではこれができないので Checkout を採用しています。また必須の引数についても説明します。``mode``は決済がどのタイプかを値に入れます。また``successUrl``も必須になります。ここには先ほど作成した決済成功時用の Dynamic Links を入れます。``cancelUrl``はオプショナルな値ですが、決済キャンセル用の Dynamic Links を値に入れます。``line_items``の値には Stripe で登録した商品のIDを入れます。

```ts
// Stripe Checkout の決済リンクを作成する関数
export const createStripeCheckoutSession = functions
  .region('asia-northeast1')
  .https.onCall(async (data, context) => {
    const customerId = data.customerId

    // ここに登録した Dynamic Links を記述
    const successUrl = 'https://sample.page.link/ABCD'
    const cancelUrl = 'https://sample.page.link/EFGH'

    try {
      const result = await stripe.checkout.sessions.create({
        payment_method_types: ['card'],
        customer: customerId,
        success_url: successUrl,
        cancel_url: cancelUrl,
        // Stripe で登録した商品の price ID をここに入れてください。
        line_items: [{ price: 'price_xxxxxxxx', quantity: 1 }],
        mode: 'subscription',
      })

      console.log('stripe.checkout.sessions.create is successful!')
      return result
    } catch (error) {
      console.log('Checkout session 作成エラー %j', error)
    }
  })
```

参考:
https://stripe.com/docs/api/checkout/sessions/create

### Firestore 上に Subscription ドキュメントを作成する関数

こちらの関数は Stripe Webhook で利用する関数であるので、Webhook が正常なものかを確かめるための関数を用意しておきます。

```ts
/// StripeのWebHookが不正なリクエストでないか確認する(Webhook で使用するため)
function verifyStripeSignature(key: String, req: any, res: any): any {
  const sig = req.headers['stripe-signature']
  let event
  try {
    // ボディのrawデータ、署名ヘッダー、署名シークレットを指定しイベントを初期化
    event = stripe.webhooks.constructEvent(req.rawBody, sig, key)
    console.log('Webhook 署名：正常')
    return event
  } catch (error) {
    // 不正なリクエストの場合
    console.log('Webhook 署名：エラー %j', error)
    return res.status(400).end()
  }
}
```

Firestore 上に Subscription ドキュメントを作成する関数のコード（以下）は多少大きめの関数にはなりますが、やっていることは以下の2つです。

1. 既に Stripe 上で Subscription が存在している場合にそれらの Subscription を解除する(後の実装上の注意で記述)
2. Firestore 上に Subscription ドキュメントを作成する

:::details Firestore 上に Subscription ドキュメントを作成する関数（TypeScript）
```ts
/// Stripe Webhook のシークレットキー(こちらは古い書き方になります。.envファイルでキーを登録すべし)
const createSubscriptionWebSec =
  functions.config().secret.stripe.webhook_secret.create_subscription

// Firestore 上に Subscription ドキュメントを作成する関数
export const createFirestoreSubscriptionDoc = functions
  .region('asia-northeast1')
  .https.onRequest(async (req, res) => {
    // 正常なリクエストかどうか確認する
    const event = verifyStripeSignature(createSubscriptionWebSec, req, res)

    try {
      // event
      console.log('event %j', event)
      // subscription
      const subscription = event.data.object
      console.log('subscription %j', subscription)
      // subscription.status
      const status = subscription.status
      console.log('subscription.status %j', status)

      const db = firebaseAdmin.firestore()

      // userId の取得
      const userQs = await db
        .collection('users')
        .where('customerId', '==', subscription.customer)
        .get()
      const userId = userQs.docs[0].id
      console.log('userId: ', userId)

      const hasAlreadySubscriptionDoc = await db
        .collection('users')
        .doc(userId)
        .collection('subscriptions')
        .where('status', '==', 'active')
        .orderBy('createdAt', 'asc')
        .get()

      // 同じ商品に対して Stripe上で 既に Subscription が存在している場合に、
      // それらの Subscription をキャンセルさせる
      console.log('hasAlreadySubscriptionDoc', hasAlreadySubscriptionDoc)
      if (hasAlreadySubscriptionDoc.docs.length >= 1) {
        hasAlreadySubscriptionDoc.docs.forEach(async (value) => {
          const subscriptionId = value.data().stripeSubscriptionId
          console.log('already-existing-subscriptionId : ', subscriptionId)
          await stripe.subscriptions.del(subscriptionId)
        })
      }

      // Firestore に Subscription ドキュメントを追加
      const subscriptionDoc = db
        .collection('users')
        .doc(userId)
        .collection('subscriptions')
        .doc()
      const docId = subscriptionDoc.id
      await subscriptionDoc.set({
        id: docId,
        stripePriceId: subscription.plan.id,
        stripeSubscriptionId: subscription.id,
        status: status,
        createdAt: firebaseAdmin.firestore.FieldValue.serverTimestamp(),
        updatedAt: firebaseAdmin.firestore.FieldValue.serverTimestamp(),
      })

      res.json({ received: true })
    } catch (error) {
      console.log('エラー : %j', error)
      res.status(500).end
    }
  })
```
:::

### Stripe の Subscription の status を Firestore に反映する関数

こちらは、Stripe の Subscription の status の変更がトリガーとなって呼ばれる関数になります。Stripe の Subscription の ID と Firestore にある``stripeSubscriptionId``を比較して一致するドキュメントの``status``を変更するようにしています。また、この関数は Firestore の Subscription ドキュメントの作成時（作成してから数秒後）に呼ばれるため、５秒の遅延を挟むようにしています。こちら詳細に関しては実装上の注意の方で説明します。

:::details Subscription の status を Firestore に反映する関数（TypeScript）
```ts
/// Stripe Webhook のシークレットキー(こちらは古い書き方になります。.envファイルでキーを登録すべし)
const updateSubscriptionWebSec =
  functions.config().secret.stripe.webhook_secret.update_subscription

// Subscription の status を Firestore に反映する関数
export const updateSubscriptionStatus = functions
  .region('asia-northeast1')
  .https.onRequest(async (req, res) => {
    // 正常なリクエストかどうか確認する
    const event = verifyStripeSignature(updateSubscriptionWebSec, req, res)

    try {
      // event
      console.log('event %j', event)
      // subscription
      const subscription = event.data.object
      console.log('subscription %j', subscription)
      // subscription.status
      const newStatus = subscription.status
      console.log('subscription.status %j', newStatus)

      // Firestore のデータを更新
      const db = firebaseAdmin.firestore()
      let subscriptionSnap = await db
        .collectionGroup('subscriptions')
        .where('stripeSubscriptionId', '==', subscription.id)
        .get()

      // Stripe の Webhook の時間差でドキュメントがない可能性があるため
      if (subscriptionSnap.empty) {
        console.log(`[${subscription.id}] subscription document is empty.`)
        await delay(5000)

        subscriptionSnap = await db
          .collectionGroup('subscriptions')
          .where('stripeSubscriptionId', '==', subscription.id)
          .get()

        if (subscriptionSnap.empty) {
          console.log(`[${subscription.id}] subscription document doesn't exist.`)
        }
      }
      const subscriptionDoc = subscriptionSnap.docs[0]
      await subscriptionDoc.ref.update({ status: newStatus })
      res.json({ received: true })
    } catch (error) {
      console.log('エラー : %j', error)
      res.status(500).end
    }
  })

// 関数の実行を遅らせるための関数
function delay(ms: number) {
  return new Promise((resolve) => setTimeout(resolve, ms))
}
```
:::

### Stripe の Subscription を解除する関数

こちらは Stripe の Subscription を解除するための関数になります。解除する際には Stripe の Subscription の ID が必要になるので、これを引数とするような``onCall``関数を使用しています。

```ts
// Stripe の Subscription を解除する関数
export const cancelStripeSubscription = functions
  .region('asia-northeast1')
  .https.onCall((data, context) => {
    const subscriptionId = data.subscriptionId
    console.log('subscriptionId : ', subscriptionId)
    return stripe.subscriptions.del(subscriptionId, {
      idempotencyKey: data.idempotencyKey,
    })
  })
```

参考:
https://stripe.com/docs/api/subscriptions/cancel

## Stripe（Webhookの設定）
[こちら](https://stripe.com/docs/webhooks/go-live)を参考に Webhook の設定を行ってください。

ブラウザ上では以下の画面から行うことができます。

![](https://storage.googleapis.com/zenn-user-upload/f2576e0da7f6-20221221.png)

設定すべき関数（エンドポイントのURL）は2つになります。以下のようにして登録を行なってください。それぞれ関数のURLは Firebase のコンソールから確認できます。

### 1. Firestore 上に Subscription ドキュメントを作成する関数（createFirestoreSubscriptionDoc）

リッスンするイベント
-　customer.subscription.created

### 2. Subscription の status を Firestore に反映する関数（updateSubscriptionStatus）

リッスンするイベント
- customer.subscription.deleted
- customer.subscription.updated


Webhook の登録後に functions の方でシークレットキーを設定してください。ここまでいったらあとはクライアント側の実装だけになります。

## Flutter(クライアント)
全体のコードは膨大なため最低限必要な部分のみ記載します。以下に、主に使用したパッケージを記載しておきます。

| パッケージ | 用途 |
| ---- | ---- |
| [hooks_riverpod](https://pub.dev/packages/hooks_riverpod) | グローバルな状態管理 |
| [flutter_hooks](https://pub.dev/packages/flutter_hooks) | ローカルな状態管理 |
| [url_launcher](https://pub.dev/packages/url_launcher) (必須) | ブラウザの起動 |
| [cloud_functions](https://pub.dev/packages/cloud_functions) (必須) | Cloud Functions の使用
| [firebase_dynamic_links](https://pub.dev/packages/firebase_dynamic_links) (必須) | Dynamic Link の受信 |


:::message
Flutter(iOS, Android) では Firebase Dynamic Links を使用するためにいくつか設定が必要です。[公式ページ](https://firebase.google.com/docs/dynamic-links/flutter/receive)を参考に設定を進めてください。
:::

### Cloud Functions の関数を呼ぶための　repository クラスを作成
Cloud Functions で作成した``onCall``関数を呼び出すためのクラスを作成します。

:::details StripeRepository
```dart
// import文は省略

final stripeRepositoryProvider = Provider.autoDispose((_) => StripeRepository());

/// Stripe関連メソッド
class StripeRepository {

  /// Create Stripe Customer and return customerId.
  Future<String?> createCustomer(String email) async {
    final callable = FirebaseFunctions.instanceFor(
      app: Firebase.app(),
      region: 'asia-northeast1',
    ).httpsCallable('stripe-createCustomer');
    final functionResult = await callable.call<Map<String, dynamic>?>({
      'email': email,
      'idempotencyKey': const Uuid().v4(),
    });
    final data = functionResult.data;
    final customerId = data?['customerId'].toString();
    return customerId;
  }

  /// Create Stripe Checkout Session and return session url.
  Future<String> createCheckoutSession({
    required String customerId,
  }) async {
    final callable = FirebaseFunctions.instanceFor(
      app: Firebase.app(),
      region: 'asia-northeast1',
    ).httpsCallable('checkout-createStripeCheckoutSession');
    final result = await callable.call<Map<String, dynamic>>({
      'customerId': customerId,
    });
    final data = result.data;
    final sessionUrl = data['url'].toString();
    return sessionUrl;
  }

  /// Cancel Stripe Subscription.
  Future<void> cancelStripeSubscription({
    required String subscriptionId,
  }) async {
    // 関数の呼び出し
    final callable = FirebaseFunctions.instanceFor(
      app: Firebase.app(),
      region: 'asia-northeast1',
    ).httpsCallable('stripe-cancelStripeSubscription');
    await callable.call<void>({
      'subscriptionId': subscriptionId,
      'idempotencyKey': const Uuid().v4(),
    });
  }
}
```
:::

### Cloud Functions の関数の呼び出し

#### ユーザ作成時
ユーザ登録をする際に customerId も Firestore に追加します。このときメールアドレスを引数にとるので注意です。

```dart
// Firestore にユーザドキュメントを作成する直前に以下を挿入
final customerId = await ref.read(stripeRepositoryProvider).createCustomer(email);
// TODO: customerId を Firestore のユーザドキュメントのフィールドに追加
```

#### サブスクリプション決済時
決済リンクを作成し、ブラウザに遷移させるコードを記述します。

```dart
// ボタンを連続で押されて複数の決済リンクが作られることを防ぐための変数です。
// こちら flutter_hooks を使用しているのでこちらもパッケージのインストールが必要です。
final isEnable = useState(true);

StreamBuilder<PendingDynamicLinkData>(
  // ここで Dynamic Links の取得を行います。
  // Firebase Dynamic Links のパッケージが必要になるのでインストールしてください。
  stream: FirebaseDynamicLinks.instance.onLink,
  builder: (context, snapshot) {
    // 決済成功時
    if (snapshot.data != null) {
      if (snapshot.data!.link.query == 'checkout_result=success') {
        return PaymentCompletionPage(); // 決済完了時の画面（好きな画面を用意してください）
      }
    }

    // ブラウザに遷移する前の画面（好きな画面を用意してください）
    return Scaffold(
      appBar: const AppBar(
        title: '決済確認ページ',
      ),
      body: Center(
        child: TextButton(
          onPressed: isEnable.value ? () {
            try {
              isEnable.value = false;

              final sessionUrl = await stripeRepo.createCheckoutSession(
                // Firestore のユーザドキュメントのクラスを用意し、customerId を取得します。
                customerId: user.customerId,
              );

              // url_launcher パッケージのインストールも必要です。
              // url_launcher のバージョンによって [forceSafariVC] の書き方が変わるので注意です。
              if (await canLaunch(sessionUrl)) {
                await launch(
                  sessionUrl,
                  forceSafariVC: false,
                );
              } else {
                throw Exception('決済ページを開けませんでした。');
              }

              // ブラウザの立ち上げにやや時間がかかるので1秒遅延させています。
              await Future<void>.delayed(const Duration(milliseconds: 1000));

            } catch (e) {
              // 適当なダイアログを用意してください。
            } finally {
              isEnable.value = true;
            }
          } : null,
          child: const Text(
            '決済に進む',
          ),
        ),
      ),
    );
```

:::message alert
iOS では、ブラウザを開く際に``forceSafariVC: false``を指定しないとうまく Dynamic Links を発火させることができません。
:::

#### サブスクリプション解除時
サブスクリプションを解除する動線を用意します。解除のための画面を用意し、必要に応じてボタンなどからサブスクリプションの解除ができるようにしましょう。

```dart
// ElevatedButton の onPressed など、サブスクリプションを解除するタイミングで以下を挿入
await ref.read(stripeRepositoryProvider).cancelStripeSubscription(subscriptionId: subscriptionId);
```

# 実装上の注意
ここでは実際に実装していて思ったこと、記事を書いている中で気づいたことを書きます。

## 1. 環境分けの対応
この実装において環境わけで注意するのは以下の2点です。

1. Firebase Dynamic Links の作成をFirebase のプロジェクトごとに行う
2. 1. にあわせてくらい Flutter（クライアント側）で環境ごとに設定を行う

詳しい説明をします。1. に関しては環境ごとに Firebase Dynamic Links を使用してリンクを作成する必要があります。具体的に、自分はアプリのドメイン名の部分の末尾に``dev``等をつけて本番環境のURLと開発環境のURLを区別しました。この設定が終わったら Cloud Functions のコードも環境ごとに変える必要があります。Firebase の環境分けには``firebase use``コマンドを使用しました。これによって Stripe の環境も変えています。決済リンクを作成する際に Dynamic Links を使用しているので下記のようにコードを加えます。

```diff ts
// Stripe Checkout の決済リンクを作成する関数
export const createStripeCheckoutSession = functions
  .region('asia-northeast1')
  .https.onCall(async (data, context) => {
    const customerId = data.customerId

+    const isProd = functions.firebaseConfig()?.projectId === '{Firebase の本番の projectId}'

+    console.log(functions.firebaseConfig()?.projectId)

+    if (isProd) {
+      console.log('本番環境')
+    } else {
+      console.log('開発環境')
+    }

+    const successUrl = isProd
+      ? 'https://sample.page.link/ABCD'
+      : 'https://sample.page.link/EFGH'
+    const cancelUrl = isProd
+      ? 'https://sampledev.page.link/ABCD'
+      : 'https://sampledev.page.link/EFGH'

-    // ここに登録した Dynamic Links を記述
-    const successUrl = 'https://sample.page.link/ABCD'
-    const cancelUrl = 'https://sample.page.link/EFGH'

    try {
      const result = await stripe.checkout.sessions.create({
        payment_method_types: ['card'],
        customer: customerId,
        success_url: successUrl,
        cancel_url: cancelUrl,
        // Stripe で登録した商品の price ID をここに入れてください。
        line_items: [{ price: 'price_xxxxxxxx', quantity: 1 }],
        mode: 'subscription',
      })

      console.log('stripe.checkout.sessions.create is successful!')
      return result
    } catch (error) {
      console.log('Checkout session 作成エラー %j', error)
    }
  })
```

ちなみに Dynamic Links を作成する際に App Store ID は開発環境の方も本番のアプリの App Store ID を入力して問題ありませんでした。

次に2. に関しては[公式ページ](https://firebase.google.com/docs/dynamic-links/flutter/receive)の設定を開発環境と本番環境それぞれやる必要があるということです。profile の変更やXcodeでの操作もあり大変ですが頑張りましょう！

## 2. サブスクリプションの一意性
基本的に Stripe では同じサブスクリプションを複数購読できてしまいます。簡単な例を出すと、Amazon Prime の会員になったときに一つのアカウントに一つでいいのに、2個も3個も登録ができてしまうということです。また Web上の決済なので、決済リンクを作成後にアプリに戻ってもう一回決済リンクを作成して、、、としていくと、何個でもブラウザ上に決済ページのタブが作成できてしまいます。正常系では異常ないのですが、異常系としてこれは対応すべきです。そのために、サブスクリプションを購読する際には同じ商品に対して既にサブスクリプションが購読されていないかを確認し、そのサブスクリプションを解除してから新たにサブスクリプションを購読するような設計にしています。それが以下のコードに該当します。加えて、決済完了後は決済できる画面に遷移しないようにすることで大幅にこの問題の発生率を減らせます。

:::details Firestore 上に Subscription ドキュメントを作成する関数（TypeScript）
```ts
/// Stripe Webhook のシークレットキー(こちらは古い書き方になります。.envファイルでキーを登録すべし)
const createSubscriptionWebSec =
  functions.config().secret.stripe.webhook_secret.create_subscription

// Firestore 上に Subscription ドキュメントを作成する関数
export const createFirestoreSubscriptionDoc = functions
  .region('asia-northeast1')
  .https.onRequest(async (req, res) => {
    // 正常なリクエストかどうか確認する
    const event = verifyStripeSignature(createSubscriptionWebSec, req, res)

    try {
      // event
      console.log('event %j', event)
      // subscription
      const subscription = event.data.object
      console.log('subscription %j', subscription)
      // subscription.status
      const status = subscription.status
      console.log('subscription.status %j', status)

      const db = firebaseAdmin.firestore()

      // userId の取得
      const userQs = await db
        .collection('users')
        .where('customerId', '==', subscription.customer)
        .get()
      const userId = userQs.docs[0].id
      console.log('userId: ', userId)

      const hasAlreadySubscriptionDoc = await db
        .collection('users')
        .doc(userId)
        .collection('subscriptions')
        .where('status', '==', 'active')
        .orderBy('createdAt', 'asc')
        .get()

      // 同じ商品に対して Stripe上で 既に Subscription が存在している場合に、
      // それらの Subscription をキャンセルさせる <-- ココ
      console.log('hasAlreadySubscriptionDoc', hasAlreadySubscriptionDoc)
      if (hasAlreadySubscriptionDoc.docs.length >= 1) {
        hasAlreadySubscriptionDoc.docs.forEach(async (value) => {
          const subscriptionId = value.data().stripeSubscriptionId
          console.log('already-existing-subscriptionId : ', subscriptionId)
          await stripe.subscriptions.del(subscriptionId)
        })
      }

      // Firestore に Subscription ドキュメントを追加
      const subscriptionDoc = db
        .collection('users')
        .doc(userId)
        .collection('subscriptions')
        .doc()
      const docId = subscriptionDoc.id
      await subscriptionDoc.set({
        id: docId,
        stripePriceId: subscription.plan.id,
        stripeSubscriptionId: subscription.id,
        status: status,
        createdAt: firebaseAdmin.firestore.FieldValue.serverTimestamp(),
        updatedAt: firebaseAdmin.firestore.FieldValue.serverTimestamp(),
      })

      res.json({ received: true })
    } catch (error) {
      console.log('エラー : %j', error)
      res.status(500).end
    }
  })
```
:::

もっと良い設計があればぜひ教えていただけると幸いです。

## 3. Web上の決済画面に遷移後、アプリを閉じる動きをした場合
これはまだクリアできていない課題になります。ブラウザ上の決済画面に遷移後にアプリを閉じる動作をユーザがすると、Dynamic Links の受信ができなくなってしまうということです。この場合にはアプリ側に Dynamic Links で戻ってきたときに決済完了画面は表示されずに決済が完了することになります。決済リンク作成後は自動でブラウザに飛ぶのであまり怒らない事象ではありますが、開発者はその点を留意しておく必要があります。

## 4. なぜ Flutter の WebView を使用しないのか
これについて、まずこの実装をしようとしたきっかけが、アプリを App Store に審査を出した際にアプリ内ではなくアプリ外（ブラウザやSMS等）で決済をするようにとリジェクトされたことです。WebView というグレーラインを攻めるよりはいっそちゃんとブラウザで決済しようと考えたからです。（無事審査は通りました）またいつか見た記事で、『WebView ではユーザが入力した情報は開発者は知ることができる』というものがあったからです。実際に自分で実装することはできませんが、そのような危険性がある以上は WebView での決済は危険であると思いこのような実装になりました。

参考:
https://gigazine.net/news/20220902-tiktok-android-vulnerability/

## 5. Cloud Functions 側のコードでもユーザやサブスクリプションの domain モデルを使った方がいい
これは実装後に思ったことです。クライアント側と同様に Cloud Functions でも domain モデルを作成して Firestore の値を扱った方がいいということです（Firestore の withConverter なども使用すべき）。今回のコードでは Firestore のドキュメントを取得するコードがかなり重複している点が目立ちます。この実装に限らず Cloud Functions を使う際はこの点を考慮してこれからコードを書いていこうと思います。

# Web上決済のメリット・デメリット
遅くなりましたがこの実装のメリット・デメリットについて解説します。全てを網羅しているわけではないのでご注意を。

- メリット
    - 手数料の低さ
    - 実装コストの低さ

- デメリット
    - 考慮すべき点が多い
    - ユーザ体験はアプリ内での決済に劣るかも
    - 一部端末でうまく動作しない
    - iOS の審査が若干運ゲー

まずはメリットについて、一番は決済手数料の低さだと思います。Apple の決済に依存しないため、詳細な比率は出しませんが、かなり決済のコストを削減できます。またカード情報の入力等の決済画面を一から作らなくて良いので実装にかかる時間も短縮できます。今回はサブスクリプションの実装をしましたが、単発決済に関してもほぼ同じ実装になるので、「早く決済機能をつけたい！」という方には良いと思います。

一方デメリットに関して、実際に実装して感じたのが低コストとはいえ考えること、やることは多いです。個人的に Dynamic Links の実装はかなり大変でした（全て自前で実装するとなるともっとやることは多いと思いますが）。あとはユーザ体験に関してです。機能としては最低限のラインを超えていると思いますが、やはりアプリ内での決済に比べたらスムーズではないかもです。とはいえカード情報はブラウザで保存しておくこともできるので、複数決済する場合でも毎回カード情報を入力しなきゃいけないわけでわないです。保存して以降は作成した決済ページであればワンクリックで決済できます。次に、一部端末でうまく動作しない、というデメリットについてです。再現性はあまりないのですが、自分の検証用の Android 実機で 決済完了後に Dynamic Links からうまく開発環境のアプリに戻って来れない、という課題がありました。機種は OPPO の少し古い型なのですが若干困ります。自分以外の開発メンバーはうまくいっていたので一応開発上の問題はなかったのです。ただこれは開発環境での問題で、本番環境の方への影響は今のところないです。最後のデメリットが1番の課題です。自分ではどうしようもないのですが、可能性として App Store の審査に落ちてしまいます。外部決済に対して多少は規約も緩くなっています。それでもリジェクトされる可能性はあります。今の段階でそういう話は聞かないのでもしかしたらの話ではあります。ここに関して、最近リジェクトされたよ、という話があれば教えていただけるとありがたいです。今のところはあまり神経質にならなくても良いと考えています。

デメリットの最後の点で不安な方は一度下の App Store の決済に関するガイドラインをチェックしておくと良いと思います。

個人的に役に立ったのは以下の記述です。投げ銭的なものは違反していないようです。

> 3.2.2 許容されない行為
> （iv）チャリティーまたは募金としてApp内で寄付金を収集する（承認を受けた非営利団体の場合や、上述のセクション3.2.1（vi）により許容される場合を除く）。そのような目的で寄付金を集めるAppはApp Storeで無料で配信する必要があり、SafariやSMSなどApp外でのみ寄付金を収集できます。

追記:
サブスクリプションに比べて、単発決済に関しては App Store の制限がきびしそうだな、とガイドラインを見て個人的に思いました。

参考:
https://developer.apple.com/jp/app-store/review/guidelines/#payments


# 終わりに
ここまで読んでいただきありがとうございます。自分が思っていたものよりもかなりボリュームのある記事になってしまいました。そのため、もしかしたら全く読まれないかもしれません。ですが、個人的にはかなり詳細に書いたつもりではあります。至らぬ点も多いかと思いますが、見つけた際にはご指摘いただけると幸いです。

2022年は Flutter をメインに触っている年でした。ハッカソンに出たり、オフラインの技術イベントに参加したりと学びの多い一年でした。また、この Zenn というサービスのおかげで快適に楽しく記事を書くことができました。開発者の皆さん、ありがとうございます！

2023年も開発者の皆さんの益々の発展を心より楽しみにしています！！

それでは良いお年を😆
