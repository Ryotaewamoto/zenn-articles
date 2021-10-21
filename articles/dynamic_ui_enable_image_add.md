---
title: "【Flutter】よくある動的に画像を追加できるUIを作成"
emoji: "🍣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", FlutterWeb, Firebase]
published: true
---

# はじめに
ご覧いただきありがとうございます。ganです。

今回は、動的に画像を追加できるUIを作る機会があったのでそれをまとめておきます。またFirebaseのfirestoreとstrorageを使用してデータのアップロードと保存することもやっていきます。

以下の記事をかなり参考にしたので合わせて読んでおくと理解しやすいかもです。

https://zenn.dev/soraef/articles/5f1bfb747a2414

# 完成品

githubにあげておきます。

以下のURLから見れます。
https://github.com/Ryotaewamoto/dynamic_ui_enable_image_add/issues/1


# 使用する技術等
Flutter2.0 (null-safety)
Flutter web
Firebase
（packageは、provider, firebaseを使います。）

pubspeck.yamlは以下の状態です。
必要に応じて追加してください。

```yaml
dependencies:
  flutter:
    sdk: flutter

  cupertino_icons: ^1.0.2
  provider: ^5.0.0

  # Firebase
  firebase_core: ^1.7.0
  firebase: ^9.0.1
  firebase_storage: ^10.0.5
  cloud_firestore: ^2.5.3
```

# 今回の内容

## step.1 domainを作成

はじめに``FavoriteImage``という名前のclassを作成します。

ここではクラス変数と2つのコンストラクタ（空のものとFirestoreのデータをいれるもの）を作成します。

空のものは画像が入っていない状態のウィジェットを作成するのに使用します。

### 補足
``documentId``はなくても大丈夫ですが、削除や編集をする場合は必要なのでとりあえず書いておきました。

```dart:favorite_image.dart
class FavoriteImage {
  String? documentId;
  String? imageURL;
  Timestamp? createdAt;

  /// ローカル変数
  int? id;

  /// コンストラクタ
  FavoriteImage({
    this.documentId,
    this.imageURL,
    this.createdAt,
    this.id,
  });

  /// インスタンスを生成
  factory FavoriteImage.create() {
    return FavoriteImage(
      id: Random().nextInt(99999),
    );
  }

  /// FireStoreからインスタンスを生成
  factory FavoriteImage.fromDoc(DocumentSnapshot doc) {
    Map<String, dynamic> data = doc.data() as Map<String, dynamic>;
    final String documentId = doc.id;
    final String imageURL = data['imageURL'];
    final Timestamp createdAt = data['createdAt'];
    return FavoriteImage(
      documentId: documentId,
      imageURL: imageURL,
      createdAt: createdAt,
    );
  }
}

```

## step.2 repositoryを作成

つぎにリポジトリを作成していきます。

ここでは、``favorite_image_repositrory``と``storage_repository``の2つ作ります。

### favorite_image_repositoryの作成

``favorite_image_repositrory``の方はFirestoreに画像を追加する、またFiresstoreから画像を読み取る処理を書いていきます。

画像のデータをFirestoreから取ってくる際に追加順にしたいのでorderByを使用しています。

全体を通して細かいエラーハンドリングはしていないのはご了承ください。

```dart:favorite_image_repository.dart
class FavoriteImageRepository {
  static FavoriteImageRepository? _instance;
  FavoriteImageRepository._();
  static FavoriteImageRepository? get instance {
    if (_instance == null) {
      _instance = FavoriteImageRepository._();
    }
    return _instance;
  }

  final _db = FirebaseFirestore.instance;

  /// 画像のリストを返す
  Future<List<FavoriteImage>> fetchFavoriteImageList() async {
    try {
      final snapshot = await _db
          .collection('favoriteImages')
          .orderBy('createdAt', descending: false) // 追加順に並ぶ
          .get();
      final favoriteImageList =
          snapshot.docs.map((doc) => FavoriteImage.fromDoc(doc)).toList();
      return favoriteImageList;
    } catch (e) {
      throw 'エラーが発生しました。\n電波の良いところで再度試してください。';
    }
  }

  /// 画像のデータ(URL)をFirestoreに追加
  Future<void> addFavoriteImage({
    required FavoriteImage favoriteImage,
  }) async {
    final newDoc = _db.collection('favoriteImages').doc();
    try {
      await newDoc.set({
        'imageURL': favoriteImage.imageURL,
        'createdAt': Timestamp.now(),
      });
    } catch (e) {
      throw 'エラーが発生しました。\n電波の良いところで再度試してください。';
    }
  }
}
```

### storage_repositoryの作成

``storage_repository``の方はFirebase storageに画像をアップロードする処理を書いていきます。

```dart:storage_repository.dart
class StorageRepository {
  static StorageRepository? _instance;
  StorageRepository._();
  static StorageRepository? get instance {
    if (_instance == null) {
      _instance = StorageRepository._();
    }
    return _instance;
  }

  final _storage = firebase.storage();

  /// 画像をアップロード
  Future<String> uploadData(String path, File file) async {
    final ref = _storage.ref().child(path);
    final snapshot = await ref.put(file).future;
    // 画像のデータをファイルからURLに直して表示
    final photoURL = await snapshot.ref.getDownloadURL();
    return photoURL.toString();
  }
}
```

## step.3 pageの作成

見た目の部分も作っていきます。

結構適当に作ったものなので必要があれば修正して使ってください。


### 補足

ファイルの名前に``first``とは言っていますが、secon以降の画面はないです。ミスリードに繋がりやすいと思ったので補足しておきます。


```dart:first_page.dart
class FirstPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider<FirstModel>(
      create: (_) => FirstModel()..init(),
      child: Consumer<FirstModel>(builder: (context, model, child) {
        return Scaffold(
          appBar: AppBar(
            title: Text('【Flutter Web】動的に画像を追加できるUIを作成'),
          ),
          body: model.isLoading
              ? Center(
                  child: CircularProgressIndicator(),
                )
              : SingleChildScrollView(
                  child: Container(
                    width: double.infinity,
                    child: Padding(
                      padding: const EdgeInsets.symmetric(
                        vertical: 32,
                        horizontal: 160,
                      ),
                      child: Column(
                        children: <Widget>[
                          Wrap(
                            alignment: WrapAlignment.start,
                            children: model.favoriteImageList
                                .map((favoriteImage) => ImageRoundedCard(
                                      favoriteImage: favoriteImage,
                                      firstModel: model,
                                    ))
                                .toList(),
                          ),
                        ],
                      ),
                    ),
                  ),
                ),
        );
      }),
    );
  }
}
```



画像を入れるウィジェットは以下のような感じです。

これも急ぎで書いたのであんまり綺麗なコードじゃないかも、、、

```dart:image_card.dart
/// 角丸カード
class ImageRoundedCard extends StatelessWidget {
  ImageRoundedCard({
    Key? key,
    required FirstModel this.firstModel,
    required FavoriteImage this.favoriteImage,
  }) : super(key: key);

  final FavoriteImage? favoriteImage;
  final FirstModel? firstModel;

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(8.0),
      child: Card(
          clipBehavior: Clip.antiAlias,
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(24),
          ),
          child: this.favoriteImage!.imageURL != null
              ? Ink.image(
                  image: NetworkImage(this.favoriteImage!.imageURL!),
                  height: 400,
                  width: 200,
                  fit: BoxFit.cover,
                )
              : InkWell(
                  onTap: () async {
                    await this
                        .firstModel!
                        .addImageAndUploadImageToStorage(this.favoriteImage!);
                  },
                  child: Container(
                    color: Colors.grey.withOpacity(0.5),
                    height: 400,
                    width: 200,
                    child: Icon(
                      Icons.add,
                      size: 40,
                    ),
                  ),
                )),
    );
  }
}
```

## step.4 modelの作成

最後にfirst_pageのもmodelの部分を書いていきます。

非同期処理があるので、isloadingという変数を用意して``init()``が行われているときはloading中の状態にしてます。



Flutter web でローカルのフォルダから画像を取得する部分は以下の記事を参考にしました。

気になる方は是非読んで見てください。

https://qiita.com/Miyaji555/items/c5f1c5dad9d9dcb2987e

### 補足

class変数の前に``_``(アンダースコア)をつけるとプライベート変数になります。プライベート変数はそのclassないでしか使用できません。ただしgetterやsetterを使うことで読み書きをできるようにすることもできます。


```dart:first_model.dart
class FirstModel extends ChangeNotifier {
  bool _isLoading = false;
  String? imageURL;
  List<FavoriteImage> favoriteImageList = [];

  FavoriteImageRepository? _favoriteImageRepository =
      FavoriteImageRepository.instance;
  StorageRepository? _storageRepository = StorageRepository.instance;

  get isLoading => _isLoading;

  Future<void> init() async {
    _startLoading();
    await _fetchFavoriteList();
    _addFavoriteImage();
    _endLoading();
  }

  /// 画像をfirestoreとstorageに保存
  Future<void> addImageAndUploadImageToStorage(
    FavoriteImage favoriteImage,
  ) async {
    _uploadImage(onSelected: (file) async {
      _startLoading();
      final path = "favoriteImage/favoriteImage${favoriteImage.id}";
      // 新しいデータを追加
      this.imageURL = await _storageRepository!.uploadData(path, file);
      _changeImageURL(favoriteImage, this.imageURL!);
      await _favoriteImageRepository!
          .addFavoriteImage(favoriteImage: favoriteImage);
      _addFavoriteImage();
      _endLoading();
    });
  }

  /// Web上でファイルを開く
  Future<void> _uploadImage({required Function(File file) onSelected}) async {
    var uploadInput = FileUploadInputElement()..accept = 'image/*';
    uploadInput.click();

    uploadInput.onChange.listen((event) {
      final file = uploadInput.files!.first;
      final reader = FileReader();
      reader.readAsDataUrl(file);
      reader.onLoadEnd.listen((event) {
        onSelected(file);
      });
    });
  }

  /// 画像のURLがfirestoreに入った時、インスタンスのimageURLを変更
  void _changeImageURL(FavoriteImage favoriteImage, String url) {
    this.favoriteImageList.forEach((element) {
      if (element.id == favoriteImage.id) {
        element.imageURL = url;
      }
    });
    notifyListeners();
  }

  /// 画像のリストをfirestoreから取得
  Future<void> _fetchFavoriteList() async {
    this.favoriteImageList =
        await _favoriteImageRepository!.fetchFavoriteImageList();
  }

  /// firestoreにデータを追加
  void _addFavoriteImage() {
    this.favoriteImageList.add(FavoriteImage.create());
  }

  void _startLoading() {
    _isLoading = true;
    notifyListeners();
  }

  void _endLoading() {
    _isLoading = false;
    notifyListeners();
  }
}
```

# まとめ

Flutterはまだまだ勉強中なので間違ってるところやアドバイス等があればコメントお願いします！！


(記事を書いていて、コードをバーーーン貼り付ける作業が多くなってしまった、、、次からは中のコードの解説ももうすこしできたらいいな〜（筆者の反省)
