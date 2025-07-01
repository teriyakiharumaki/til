# Turboって何？

## ✅ 行ったこと

- Turboについて調べた
- Hotwireについて調べた

Turboを学ぶ上でHotwireは必須なので、まずHotwireから

## Hotwireとは

- Hotwireとは、Rails7からRailsのデフォルトになった、モダンなWebアプリケーションを作るための新しいアプローチ。[1]<br>
もう少し、詳しく説明すると、
サーバーサイドで生成されたHTMLを直接ブラウザに送信し、JavaScriptを最小限に抑えて、より高速でインタラクティブなウェブアプリケーションを構築することができるRuby on Railsのフレームワーク。<br>
SPAではないが、React等のjsライブラリを使用せずに、SPA風のUXを実現することができるRailsのフレーワークというイメージだろうか。<br>
Hotwireは、主に３つのコンポーネントで構成されている。<br>
  - Turbo
  - Stimulus
  - Strada

- HotwireはRails7からRailsのデフォルトになった、モダンなWebアプリケーションを作るための新しいアプローチだよ。HotwireはTurboとStimulusとStradaという3つの技術から構成されるよ。[2]<br>
  - Turbo
  - Stimulus
  - Strada

- Hotwireは、Ruby on Railsのエコシステムにおいて、フロントエンドとサーバーサイドの連携を効率化するためのツールセットです。Hotwireは、主にTurboとStimulusの2つのライブラリから成り立っており、Turboはページ遷移や動的な更新を最適化し、Stimulusはインタラクションを管理するために使われます。[3]

- Hotwire（HTML Over The Wire）は、Turbo と Stimulus という 2 つのライブラリで構成される、シンプルで高速な Web アプリ開発を目指すフレームワーク。<br>
Rails7 からは、従来の Webpacker ではなく、Hotwire と Importmap で JavaScript を管理するようになっている。<br>
Hotwire は Rails に依存しているわけではないので、Rails 以外でも使える。[4]

## Turboとは

- Turboは、htmlのbody タグのみ置換するSPA風のアプリケーションを作成することができるHotwireのコンポーネントの１つ。[1]<br>
Turboは４つの要素がある。<br>
  - Turbo Drive
  - Turbo Frames
  - Turbo Streams
  - Turbo Native

- Turboを使うとJavaScriptを書かずにSPA風のアプリケーションを実現できるようになるよ。[2]

- Turboは、ページ遷移を最適化し、従来のAJAXのようにページ全体を更新することなく、必要な部分だけを更新する仕組みです。これにより、ユーザー体験が向上し、ページ遷移やリロードを高速化することができます。[3]

- Turbo とは、Hotwire によって提供される、ページ全体のリロードなしでページ更新を行う仕組み。<br>
従来の Rails アプリであれば JavaScript や Ajax が必要だった非同期のページ更新を、ほとんど JavaScript を使わずに実現できる。[4]

## つまり、Turboとは

Turboは、HotwireというRails標準のフロントエンドフレームワークの一部で、SPA（シングルページアプリケーション）のような体験を提供する仕組みです。<br>
ページ全体をリロードせずに、画面の一部だけを動的に更新できるのが大きな特徴です。<br>
従来のJavaScriptフレームワークのように複雑な設定や構成を必要とせず、Railsの規約に沿ってコードを書くことで、バックエンドとフロントエンドをシームレスに連携できます。<br>
JavaScriptを最小限に抑えながら、ユーザーにとってスムーズで高速な操作感を実現できるのが魅力です。

## 参考サイト
[1](https://qiita.com/Ninomin/items/d0f7e0c2c80b49a7596c)<br>
[2](https://zenn.dev/shita1112/books/cat-hotwire-turbo/viewer/abstract)<br>
[3](https://www.next.inc/articles/2025/rails%E3%81%AEhotwire%E3%81%A8turbo%E3%81%A8%E3%81%AF%EF%BC%9F.html)<br>
[4](https://keita1899.hatenablog.com/entry/2024/12/21/010002)<br>