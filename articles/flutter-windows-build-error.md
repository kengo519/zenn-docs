---
title: "【Flutter】windowsアプリのビルドに失敗した時の対処法"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter]
published: true
---
## はじめに
Flutterを触ってみたくなったので以下を参考に環境構築をしていました。
https://zenn.dev/kazutxt/books/flutter_practice_introduction/viewer/06_chapter1_environment
こちらの本はわかりやすく丁寧に説明されているのでオススメです。
今回、windowsアプリのビルドに躓いてしまったので、メモとして残しておきます。

## エラーと対処方法
VS Codeの`Run and Debug`を実行したときに以下のエラーが出力されました。
![](/images/flutter-windows-build-error/error.png)
フォルダパスが文字化けしていることがわかります。
日本語を含まないフォルダパスにプロジェクトを作成することで、エラーが解消されました！
