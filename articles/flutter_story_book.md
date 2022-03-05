---
title: "【Flutter】storybookを使ってWidget開発" # 記事のタイトル
emoji: "📕" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Storybook"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに
ご覧いただきありがとうございます。ganです。
今回はstorybook というパッケージを紹介していこうと思います。

# storybookとは

”Webアプリケーションの”UIコンポーネントの管理・テストをすることが出来るオープンソースツールです。特に、React、Vue、Angularなどの主要なJSフレームワークで導入できるので利用範囲も広いです。

Webアプリケーション開発では有名っぽいのですが、実はFlutterでもstorybookが簡単に使える便利なパッケージがあります。

最近ではFlutter Webも盛り上がってきているので需要がありそうです。

https://pub.dev/packages/storybook_flutter

また以下のURLからWeb開発をする場合にどんな感じか、Demoアプリで確かめることができます。実際に触ってから記事を読むと理解が深まると思います。ぜひ。

https://ookami-kb.github.io/storybook_flutter/#/

以下ではその具体的な使い方について書いていきます。

# 使用例（ios）

見辛くてすいません。Simulatorのgifです。

Storyの切り替えやknobsの変更（テキスト、アップバーの影と色、Widgetの数を増加、FABの有無）、ダークモードへの変更や端末の種類を変更したりしています。

![Simulator Screen Recording - iPhone 11 - 2022-03-05](https://user-images.githubusercontent.com/75112184/156879862-9806b510-46fb-484a-9b58-b51b0ff914b2.png)

以下のコードをfluter project作成後、main.dartにコピペすれば同じように動くはずです。

# サンプルコード（全体）

やや長いですが、全体像としてコードを載せておきます。
後半にStorybook、Storyというクラスごとに解説していきます。

```dart
import 'package:flutter/material.dart';
import 'package:storybook_flutter/storybook_flutter.dart';

void main() {
  runApp(const MyApp());
}

final List<Plugin> _plugins = initializePlugins( // iosやandroid端末でstorybookを使用する場合は上2つをfalseにするのがおすすめです。Webページであれば画面サイズが大きのでサイドパネルを表示しても収まりますが、スマホのサイズではサイドパネルを表示する幅が確保できないからです
  contentsSidePanel: false,
  knobsSidePanel: false,
  initialDeviceFrameData: DeviceFrameData(
    device: Devices.ios.iPhone12,
  ),
);

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Storybook(
            initialStory: 'Screens/Default',
            plugins: _plugins, 
            showPanel: true,
            stories: [
              Story(
                name: 'Screens/Default',
                description: 'Story with default flutter app',
                builder: (context) => Scaffold(
                  appBar: AppBar(
                    title: Text(
                      context.knobs.text(
                        label: 'Title',
                        initial: 'AppBarTitle',
                        description: 'The title of the app bar.',
                      ),
                    ),
                    elevation: context.knobs.slider(
                      label: 'AppBar elevation',
                      initial: 4,
                      min: 0,
                      max: 10,
                      description: 'Elevation of the app bar.',
                    ),
                    backgroundColor: context.knobs.options(
                      label: 'AppBar color',
                      initial: Colors.blue,
                      description: 'Background color of the app bar.',
                      options: const [
                        Option(
                          label: 'Blue',
                          value: Colors.blue,
                          description: 'Blue color',
                        ),
                        Option(
                          label: 'Green',
                          value: Colors.green,
                          description: 'Green color',
                        ),
                      ],
                    ),
                  ),
                  body: SizedBox(
                    width: double.infinity,
                    child: Column(
                      mainAxisAlignment: MainAxisAlignment.center,
                      crossAxisAlignment: CrossAxisAlignment.center,
                      children: List.generate(
                        context.knobs.sliderInt(
                          label: 'Items count',
                          initial: 2,
                          min: 1,
                          max: 5,
                          description: 'Number of items in the body container.',
                        ),
                        (_) => const Padding(
                          padding: EdgeInsets.all(8),
                          child: Text('Hello World!'),
                        ),
                      ),
                    ),
                  ),
                  floatingActionButton: context.knobs.boolean(
                    label: 'FAB',
                    initial: true,
                    description: 'Show FAB button',
                  )
                      ? FloatingActionButton(
                          onPressed: () {},
                          child: const Icon(Icons.add),
                        )
                      : null,
                ),
              ),
              Story(
                name: 'Screens/Counter',
                description: 'Demo Counter app with about dialog.',
                builder: (context) => CounterPage(
                  title: context.knobs.text(label: 'Title', initial: 'Counter'),
                  enabled: context.knobs.boolean(label: 'Enabled', initial: true),
                ),
              ),
              Story(
                name: 'Widgets/Text',
                description: 'Simple text widget.',
                builder: (context) => const Center(child: Text('Simple text')),
              ),
            ],
    );
  }
}

class CounterPage extends StatefulWidget {
  const CounterPage({
    Key? key,
    required this.title,
    this.enabled = true,
  }) : super(key: key);

  final String title;
  final bool enabled;

  @override
  _CounterPageState createState() => _CounterPageState();
}

class _CounterPageState extends State<CounterPage> {
  int _counter = 0;

  void _incrementCounter() => setState(() => _counter++);

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(
          title: Text(widget.title),
          actions: [
            IconButton(
              icon: const Icon(Icons.help),
              onPressed: () => showAboutDialog(
                context: context,
                applicationName: 'Storybook',
                applicationVersion: '0.0.1',
                applicationIcon: const Icon(Icons.book),
                applicationLegalese: 'MIT License',
              ),
            ),
          ],
        ),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const Text('You have pushed the button this many times:'),
              Text(
                '$_counter',
                style: Theme.of(context).textTheme.headline4,
              ),
            ],
          ),
        ),
        floatingActionButton: widget.enabled
            ? FloatingActionButton(
                onPressed: _incrementCounter,
                tooltip: 'Increment',
                child: const Icon(Icons.add),
              )
            : null,
      );
}
```

## パッケージ内のクラスの使い方

### Storybook

storybookは、一冊の本のイメージです。中にstoryというページがいくつか含まれている感じです。
なのでstorybookクラスの引数には本の性質を入れてあげます。何を最初のページにするのか、本の大きさ（スマホのサイズ）はいくつかといった具合です。

またStorybookクラスでは、materialWrapperという関数が呼ばれます。なので上からMaterialAppで囲う必要はありません。materialじゃなくて``cupertino``を使いたい！という場合は、Storybookクラスの引数で```wrapperBuilder: cupertinoWrapper```と指定するとできます。

```dart
Widget materialWrapper(BuildContext context, Widget? child) => MaterialApp(
      theme: ThemeData.light(),
      darkTheme: ThemeData.dark(),
      debugShowCheckedModeBanner: false,
      useInheritedMediaQuery: true,
      home: Scaffold(body: Center(child: child)),
);
```

```dart
final List<Plugin> _plugins = initializePlugins(
  contentsSidePanel: true, // storyを選択するサイドパネルの表示
  knobsSidePanel: true, // knobsを変更するサイドパネルの表示
  initialDeviceFrameData: DeviceFrameData(
    device: Devices.ios.iPhone12,
  ), // この場合はiPhone12を初期表示
);

return Storybook(
  wrapperBuilder: materialWrapper,
  initialStory: 'Screens/Default', // 画面生成時に初期に表示したいstoryのnameを指定
  plugins: _plugins, // サイドパネルの表示位置や画面生成時のデバイスを設定
  stories: [
    Story(), // 確認したいWidgetをStoryに登録
    Story(),
    Story(), 
  ],
);

```

### Story

storyはstorybookという本に含まれる各ページのイメージです。
``context.knobs``プロパティのメソッド（text, slider, sliderInt, options, boolean）を使うことで,
その値を端末上で変更でき、また変更した際のWidgetの見た目を画面で確認することができます。
また使用箇所も明記しておきます。

```dart
Story(
  name: 'Screens/Default',
  description: 'Story with default flutter app',
  builder: (context) => Scaffold(
    appBar: AppBar(
      title: Text(
        context.knobs.text( // 使用例1 text(String)
          label: 'Title',
          initial: 'AppBarTitle',
          description: 'The title of the app bar.',
        ),
      ),
      elevation: context.knobs.slider( // 使用例2 slider(double)
        label: 'AppBar elevation',
        initial: 4,
        min: 0,
        max: 10,
        description: 'Elevation of the app bar.',
      ),
      backgroundColor: context.knobs.options( // 使用例3 options(List<Option>) ColorやIcon等
        label: 'AppBar color',
        initial: Colors.blue,
        description: 'Background color of the app bar.',
        options: const [
          Option(
            label: 'Blue',
            value: Colors.blue,
            description: 'Blue color',
          ),
          Option(
            label: 'Green',
            value: Colors.green,
            description: 'Green color',
          ),
        ],
      ),
    ),
    body: SizedBox(
      width: double.infinity,
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        crossAxisAlignment: CrossAxisAlignment.center,
        // List.generateを使うことで指定した個数のWidgetを表示することができます。
        children: List.generate(
          context.knobs.sliderInt( // 使用例4 sliderInt(int)
            label: 'Items count',
            initial: 2,
            min: 1,
            max: 5,
            description: 'Number of items in the body container.',
          ),
          (_) => const Padding(
            padding: EdgeInsets.all(8),
            child: Text('Hello World!'),
          ),
        ),
      ),
    ),
    floatingActionButton: context.knobs.boolean( // 使用例 boolean(bool)
      label: 'FAB',
      initial: true,
      description: 'Show FAB button',
    )
        ? FloatingActionButton(
            onPressed: () {},
            child: const Icon(Icons.add),
          )
        : null,
  ),
),

```

:::message
Storybookの引数initialStoryやStoryの引数nameでは値に``/``を一つのみ含むことができます。2つ以上も書けるのですが、二つ目の``/``以降は読み取られません。ご注意を。
:::

# 最後に
まだ試してはいませんがアニメーションを作るのにも役に立つと思います。アニメーションの間隔や何秒継続するかを``sliderInt``を使ってあげればベストなものが作れそうです。

Flutterにはホットリロードがあるので使える場所は多少限定的にはなると思いますが、面白いパッケージですのでぜひ使ってみてください。

皆様のアプリ開発ライフに役立つことができれば幸いです。
