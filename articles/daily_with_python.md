---
title: "Github日記 ~Pythonでファイルを生成しながら~" # 記事のタイトル
emoji: "🎍" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Python", "markdown"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに
ご覧いただきありがとうございます。ganです。

今回はPythonを使ってGithub管理するTODOリスト(日記的なもの)を作ってみました。

作ってみようと思ったのは以下の記事を見たからです。ぜひ見てみてください！

https://zenn.dev/hand_dot/articles/85c9640b7dcc66

この記事とは少し違いますがVSCodeで記録が書けたらいいなと思ったのPythonを調べながらやってみました。初めてPythonを使ってみたのであまりPythonの良さを発揮できていないと思いますが、少しでも興味を持っていただけると幸いです。


# 作ったもの

今回作ったのは、MarkDown形式のファイルを自動で生成するものです。.mdファイルの中身はTODOリストのようになっていますが、必要に応じて書き換えられます。また同名のファイルが存在した場合、既存のファイルには変更を加えない仕様にしてあります。フォルダが存在しない場合には自動でフォルダも作成するようにしました。

## ファイルを作成
![](https://storage.googleapis.com/zenn-user-upload/ad78ac445e9d-20220101.gif)

見づらくてすいません🙇‍♀️
右上のボタンでcreate.pyと言う名前のファイルを実行しています。
再度押すと、「既に作成済みです。」と言うメッセージがターミナルに表示されます。

## 作成したファイルの中身
![](https://storage.googleapis.com/zenn-user-upload/569638b74d65-20220101.png)

# 開発環境
エディターはVSCodeを使用しています。
Pythonのバージョンは3.8.5です。

VSCodeの拡張機能として『Python』を入れておくとファイルを開いた時に右上のボタンからから実行できるのでインストールしておいてください。


# 使用したコード全体
```py:create.py
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
import os

import datetime

# today (今回は使用していない)
dt_now = datetime.datetime.now()
year = dt_now.year
month = dt_now.month
day = dt_now.day

# tomorrow
tomorrowYear = (dt_now + datetime.timedelta(days = 1)).year
tomorrowMonth = (dt_now + datetime.timedelta(days = 1)).month
tomorrowDay = (dt_now + datetime.timedelta(days = 1)).day

# .mdファイルを作成(既にある場合は何もしない)
def write_md(file_name, myCheck):
    if not myCheck:   # notを付与することで、Falseの場合に実行（真(True)でない）
        with open(file_name, 'w') as f:
            f.write('\
# {0}{1}{2}\n\
## やること\n\
- [ ] \n\
- [ ] \n\
- [ ] \n\
\n\
## 反省点\n\
- \n\
- \n\
- \n\
\n\
## その他\n\
- \n\
'.format(tomorrowYear, tomorrowMonth, tomorrowDay))
            f.close()
            print('🔥created new file!')
    else: # Trueの場合
        print('🌟you have already created.')

# フォルダを作成(既にある場合は何もしない)
def make_directory(path):
    if not os.path.isdir(path):
        os.makedirs(path)
        print(path + 'folder does not exist.')
        print(path + ' folder is created!')

def main():
    myCheck = os.path.isfile(file_path) # 検索ファイルがあればTrueを返し、なければFalseを返す（ブール型boolと呼ぶ）。
    print('Whether to exist the file:' + str(myCheck))

    write_md(file_name, myCheck) # witeTxt関数へ、変数を引数に渡して実行

if __name__ == '__main__':
    ### parameter ###
    current_path = os.getcwd()  # パス設定（カレントディレクトリ）
    make_directory('{0}{1}'.format(tomorrowYear, tomorrowMonth))

    file_name = '{0}{1}/{0}{1}{2}.md'.format(tomorrowYear, tomorrowMonth, tomorrowDay)  # 検索ファイル名を設定
    file_path = os.path.join(current_path, file_name) # パスとファイル名を結合
    print(file_path)
    
    ### call function ###
    main()

```

# コードの解説

少しだけ解説します。

1. 現在の日付の取得（明日の日付も取得）

```py
import datetime

# today (今回は使用していない)
dt_now = datetime.datetime.now()
year = dt_now.year
month = dt_now.month
day = dt_now.day

# tomorrow
tomorrowYear = (dt_now + datetime.timedelta(days = 1)).year
tomorrowMonth = (dt_now + datetime.timedelta(days = 1)).month
tomorrowDay = (dt_now + datetime.timedelta(days = 1)).day
```

この部分で現在時刻を取得しています。
``datetime.datetime.now()``で今日の日付を取得し、1日ずらした値をtomorrow~という変数名で定義しています。TODOなので明日の予定を表示できたらいいなと思い、この仕様にしました。もちろん今日の日付を使っても問題ないです。

2. ファイルの作成

```py
# .mdファイルを作成(既にある場合は何もしない)
def write_md(file_name, myCheck):
    if not myCheck:   # notを付与することで、Falseの場合に実行（真(True)でない）
        with open(file_name, 'w') as f:
            f.write('\
# {0}{1}{2}\n\
## やること\n\
- [ ] \n\
- [ ] \n\
- [ ] \n\
\n\
## 反省点\n\
- \n\
- \n\
- \n\
\n\
## その他\n\
- \n\
'.format(tomorrowYear, tomorrowMonth, tomorrowDay))
            f.close()
            print('🔥created new file!')
    else: # Trueの場合
        print('🌟you have already created.')
```

ファイルを作成する関数を定義しています。先ほど取得した日付を引数にしてファイル内に書き加えています。myCheckというbool値によってファイルを作成するか、既にある場合には変更を加えないかで条件分岐しています。

writeメソッドの引数にファイル内に加えたいコード（もしくはテキスト）を入力してください。

3. 既にフォルダがある場合の処理

```py
# フォルダを作成(既にある場合は何もしない)
def make_directory(path):
    if not os.path.isdir(path):
        os.makedirs(path)
        print(path + 'folder does not exist.')
        print(path + ' folder is created!')
```

ここではファイルを作成する際にそのフォルダがなかった時の処理を書いています。


4. main関数の定義と呼び出し

```py
def main():
    myCheck = os.path.isfile(file_path) # 検索ファイルがあればTrueを返し、なければFalseを返す
    print('Whether to exist the file:' + str(myCheck))

    write_md(file_name, myCheck) # wite＿md関数へ、変数を引数に渡して実行

if __name__ == '__main__':
    ### parameter ###
    current_path = os.getcwd()  # パス設定（現在のディレクトリ）
    make_directory('{0}{1}'.format(tomorrowYear, tomorrowMonth))

    file_name = '{0}{1}/{0}{1}{2}.md'.format(tomorrowYear, tomorrowMonth, tomorrowDay)  # 検索ファイル名を設定
    file_path = os.path.join(current_path, file_name) # パスとファイル名を結合
    print(file_path)
    
    ### call function ###
    main()
```

最後に1~3のコードを呼び出しています。

# まとめ

初めてPythonに触れたのでちゃんとできているかはわかりませんが、個人的には満足の行くものができました。

今回はMarkDown形式のファイルを作成しましたが、他のファイルの作成など結構応用が効きそうです。

よければ使ってみてください！

# 参考記事

- 【Python】ファイルの作成と読み込み open,write,read,with,seek

https://code-graffiti.com/create-and-load-file-in-python/#toc1

- 【EC2】エラー：Non-ASCII character '\xe8'の対処法

https://qiita.com/shizen-shin/items/5451840f7946f8db585e

- Pythonで現在時刻・日付・日時を取得

https://note.nkmk.me/python-datetime-now-today/

- 【Python】文字列に変数を入れて文字列を動的に変更させる（format、f_strings）

https://office54.net/python/basic/python-format-fstrings

- Pythonで長い文字列を複数行に分けて書く

https://note.nkmk.me/python-long-string/

- Visual Studio Code で Markdown 編集環境を整える

https://qiita.com/kumapo0313/items/a59df3d74a7eaaaf3137#3-markdown-checkbox 

- Python 指定ファイルの存在をチェックし、無ければファイルを作成する。有れば追記する。

https://hk29.hatenablog.jp/entry/2018/05/01/112953

- Pythonでディレクトリ（フォルダ）を作成するmkdir, makedirs

https://note.nkmk.me/python-os-mkdir-makedirs/
