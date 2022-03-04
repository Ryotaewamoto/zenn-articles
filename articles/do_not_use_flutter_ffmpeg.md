---
title: "【Flutter】flutter_ffmpegは使わない方がよさそう" # 記事のタイトル
emoji: "🩹" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに

ご覧いただきありがとうございます。ganです。
今回はflutter_ffmpeg というパッケージでつまったので記事にまとめておきます。
日本語の記事等があまりなかったので誰かの参考になれば幸いです。


# 実際に直面したエラー

flutterのdebugモードでビルドしようとした際に以下のエラー

```
Launching lib/main.dart on iPhone 11 in debug mode...
Xcode build done.                                           31.0s
Failed to build iOS app
Error output from Xcode build:
↳
    ** BUILD FAILED **
Xcode's output:
↳
    Writing result bundle at path:
    	/var/folders/06/xz9x3dpx3j1c8khjw5y1xnr80000gn/T/flutter_tools.sy49WF/flutter_ios_build_temp_dirylJrRz/temporary_xcresult_bundle
In file included from /Users/ryotaiwamoto/Developer/flutter/.pub-cache/hosted/pub.dartlang.org/flutter_ffmpeg-0.4.2/ios/Classes/FlutterFFmpegPlugin.m:20:
/Users/ryotaiwamoto/Developer/flutter/.pub-cache/hosted/pub.dartlang.org/flutter_ffmpeg-0.4.2/ios/Classes/EmptyLogDelegate.h:20:10: fatal error: 'mobileffmpeg/LogDelegate.h' file not found
    #include <mobileffmpeg/LogDelegate.h>
             ^~~~~~~~~~~~~~~~~~~~~~~~~~~~
    1 error generated.
In file included from /Users/ryotaiwamoto/Developer/flutter/.pub-cache/hosted/pub.dartlang.org/flutter_ffmpeg-0.4.2/ios/Classes/FlutterExecuteDelegate.m:20:
/Users/ryotaiwamoto/Developer/flutter/.pub-cache/hosted/pub.dartlang.org/flutter_ffmpeg-0.4.2/ios/Classes/FlutterExecuteDelegate.h:21:9: fatal error: 'mobileffmpeg/ExecuteDelegate.h' file not found
    #import <mobileffmpeg/ExecuteDelegate.h>
            ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    1 error generated.
In file included from /Users/ryotaiwamoto/Developer/flutter/.pub-cache/hosted/pub.dartlang.org/flutter_ffmpeg-0.4.2/ios/Classes/EmptyLogDelegate.m:20:
/Users/ryotaiwamoto/Developer/flutter/.pub-cache/hosted/pub.dartlang.org/flutter_ffmpeg-0.4.2/ios/Classes/EmptyLogDelegate.h:20:10: fatal error: 'mobileffmpeg/LogDelegate.h' file not found
    #include <mobileffmpeg/LogDelegate.h>
             ^~~~~~~~~~~~~~~~~~~~~~~~~~~~
    1 error generated.
    note: Using new build system
    note: Planning
    note: Build preparation complete
    note: Building targets in dependency order
    Result bundle written to path:
    	/var/folders/06/xz9x3dpx3j1c8khjw5y1xnr80000gn/T/flutter_tools.sy49WF/flutter_ios_build_temp_dirylJrRz/temporary_xcresult_bundle
Lexical or Preprocessor Issue (Xcode): 'mobileffmpeg/LogDelegate.h' file not found
/Users/ryotaiwamoto/Developer/flutter/.pub-cache/hosted/pub.dartlang.org/flutter_ffmpeg-0.4.2/ios/Classes/EmptyLogDelegate.h:19:9

Lexical or Preprocessor Issue (Xcode): 'mobileffmpeg/ExecuteDelegate.h' file not found
/Users/ryotaiwamoto/Developer/flutter/.pub-cache/hosted/pub.dartlang.org/flutter_ffmpeg-0.4.2/ios/Classes/FlutterExecuteDelegate.h:20:8

Could not build the application for the simulator.
Error launching application on iPhone 11.
Exited (sigterm)
```

flutter_ffmpegってやつが悪さしてるっぽい。


# 参考になった記事

https://flutterrepos.com/lib/tanersener-flutter-ffmpeg

自分の場合、この記事のCommentsの15（記事のかなり下の方）

15. fatal error: 'mobileffmpeg/LogDelegate.h' file not found

に当てはまってました（「'mobileffmpeg/LogDelegate.h'なんてファイルねーよ」って意味です）。

どうやらこの記事のタイトル『Flutter-ffmpeg - FFmpeg plugin for Flutter. Not maintained anymore. Superseded by FFmpegKit.』曰く、

flutter_ffmpegは更新されてないから、ffmpeg_kit_flutterを使え！

ということです。

https://pub.dev/packages/ffmpeg_kit_flutter/install

なので、いったんこのflutter_ffmpegパッケージをpubspec.yamlから消してpub getしてからビルドし直したらいけました！


# まとめ
ビルドのエラーってめちゃくちゃ辛いですよね。コード書けないから開発は進まないし時間の無駄感がすごい。エラー文を見て、stack overflow見て、他の人の記事みて、、、ってやるしかないんですけどね。一流のエンジニアになるための道のりだと思って一緒に頑張りましょう！
