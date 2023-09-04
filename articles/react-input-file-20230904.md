---
title: "Reactで<input type=\"file\">を使ってみた"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [react]
published: false
---

Reactで<input type="file">を使う機会があり、onChangeイベントのコールバックを受ける際に手こずったので調べた内容をまとめました。

本記事ではReact + Typescriptを前提とします。

## ファイル読み込み
まずはinput要素でファイル読み込みのコードを実装します。今回はJSONファイルの読み込みをしたかったので、accept属性に.jsonを指定しています。

```tsx
import React from 'react';

export const App: React.FC = () => {
  return (
    <input type="file" accept='.json'/>
  );
}
```
accept='.json'とすることで、ファイル選択時にJSONファイルのみが選択できるようになります。
しかし、**ファイル選択ダイアログのプルダウンから「すべてのファイル(*.*)」を選択されてしまうと自由にファイルを選択できてしまいます。**

特定の形式のみのファイルを受け付けて処理をしたい場合は、ファイル選択時のコールバックを受けて要求したファイル形式であるかをチェックする必要があります。

## ファイル選択時のコールバック
ファイル選択時のコールバックはonChangeイベントで受けることができます。
厳密には**選択したファイルに変更があった時**にonChangeに指定した処理が実行されます。なので連続して同じファイルを選択してもonChangeイベントは発火しません。

```tsx
import React from 'react';

export const App: React.FC = () => {
  return (
    <input type="file" accept='.json' onChange={(event: React.ChangeEvent<HTMLInputElement>) => {
      const files = event.currentTarget.files;
      // ファイルがなければ終了
      if (!files || files?.length === 0) return;
      // 先頭のファイルを取得
      const file = files[0];
      console.log(file.name);
    }} />
  );
}
```
ファイル選択ダイアログでは複数のファイルを選択可能であり、`event.currentTarget.files`は**選択されたファイル群**を示します。上記コードでは、選択したファイル群の先頭要素をfileに代入し、ファイル名をコンソール出力します。

実際は、選択されたファイルがJSONファイルであるか否かを判別したいので、フック(setState)を使用してファイル名を受け、判別関数に渡します。

```tsx
import React, { useState } from 'react';

const isJSON = (name: string): boolean => {
  return "json" === name.split(".").at(-1);
}

export const App: React.FC = () => {
  const [name, setName] = useState<string>();

  return (
    <React.Fragment>
      <input type="file" accept='.json' onChange={(event: React.ChangeEvent<HTMLInputElement>) => {
        const files = event.currentTarget.files;
        // ファイルがなければ終了
        if (!files || files?.length === 0) return;
        // 先頭のファイルを取得
        const file = files[0];
        setName(file.name);
      }} />
      <br />
      {name ? isJSON(name) ? "JSONファイルです。" : "JSONファイルではありません。" : null}
    </React.Fragment>
  );
}
```

以下の箇所が三項演算子を使っているため理解しにくいですが、ファイル未選択時は何も表示せず、ファイル選択時はJSONファイルか否かを表示します。
```tsx
{name ? isJSON(name) ? "JSONファイルです。" : "JSONファイルではありません。" : null}
```

##　ファイルの内容を取得する

ファイルの内容を取得する場合は`FileReader`を使用します。
```tsx
import React, { useState } from 'react';

const isJSON = (name: string): boolean => {
  return "json" === name.split(".").at(-1);
}

export const App: React.FC = () => {
  const [name, setName] = useState<string>();
  const [contents, setContents] = useState<string>();

  return (
    <React.Fragment>
      <input type="file" accept='.json' onChange={(event: React.ChangeEvent<HTMLInputElement>) => {
        const files = event.currentTarget.files;
        // ファイルがなければ終了
        if (!files || files?.length === 0) return;
        // 先頭のファイルを取得
        const file = files[0];
        setName(file.name);

        const reader = new FileReader();
        // ファイル読み込み完了時に発火するリスナー
        reader.addEventListener("load", () => {
          setContents(typeof reader.result === "string" ? reader.result : undefined);
          console.log(contents);
        });
        reader.readAsText(file, 'UTF-8');
      }} />
      <br />
      {name ? isJSON(name) ? "JSONファイルです。" : "JSONファイルではありません。" : null}
    </React.Fragment>
  );
}
```
あらかじめ`addEventListener`でファイル読み込み完了時の処理を登録しておき、`readAsText`でfile読み込みが完了した時にその処理が実行されます。
上記コードでは、読み込んだ内容をcontentsに代入し、コンソールに出力します。

## 任意の要素をクリックしたときにファイル選択ダイアログを呼び出す
`<input type="file">`はボタンと選択したファイル名が表示されますが、これらを任意のデザインに変更したい場合があるかと思います。

input要素を非表示(display:none)にし、任意要素のonClickイベントでinput要素のクリックイベントを呼び出すことで実現できます。

```css:InputFile.css
.input-file {
  display: none;
}
```
```tsx:InputFile.tsx
import React, { useState } from 'react';
import './InputFile.css';

const isJSON = (name: string): boolean => {
  return "json" === name.split(".").at(-1);
}

export const App: React.FC = () => {
  const [name, setName] = useState<string>();

  return (
    <React.Fragment>
      <input id="input-file" type="file" accept='.json' onChange={(event: React.ChangeEvent<HTMLInputElement>) => {
        const files = event.currentTarget.files;
        // ファイルがなければ終了
        if (!files || files?.length === 0) return;
        // 先頭のファイルを取得
        const file = files[0];
        setName(file.name);

        const reader = new FileReader();
        // ファイル読み込み完了時に発火するリスナー
        reader.addEventListener("load", () => {
          setContents(typeof reader.result === "string" ? reader.result : undefined);
          console.log(contents);
        });
        reader.readAsText(file, 'UTF-8');
      }} />
      <br />
      <button onClick={(): void => {
        const element = document.getElementById("input-file");
        element?.click();
      }}>
        ボタン
      </button>
      {name ? isJSON(name) ? "JSONファイルです。" : "JSONファイルではありません。" : null}
    </React.Fragment>
  );
}
```
正直なところ、input要素を非表示にするのは無理やりな感じがしています。他に良い方法をご存じの方がいましたらコメントいただけたら幸いです。

## おわりに
以上、React + <input type="file">の使用方法を実装例とともに説明しました。

本記事のソースコードは以下で公開しているので参考にしてみてください。https://github.com/kengo519/react-input-file

コードをクローンした後はnpm installを実行してください。

また、GitHubでは静的なサイトホスティングサービスGitHub Pagesでウェブサイトを公開することができます。今回の紹介したコードのWebページをGitHub Pagesを使って公開したので、以下で動作確認することができます。
https://kengo519.github.io/react-input-file/
