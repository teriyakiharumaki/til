# javaのインストール

## ✅ 行ったこと

- Spring Initializr を使って、Spring Bootプロジェクトを作成した

## 参考にしたサイト
https://route-zero.com/recruit/route/922/

## Spring Initializr を使って、Spring Bootプロジェクトを作成
設定内容

- Project: Maven

- Language: Java

- Spring Boot Version: 最新の安定版

- Dependencies:

  - Spring Web

  - Thymeleaf

  - Spring Boot DevTools

## サイトで実際に入力した画面
[![Image from Gyazo](https://i.gyazo.com/448b38c9017c56f7d708548494b76c78.png)](https://gyazo.com/448b38c9017c56f7d708548494b76c78)

GENERATEを押すと、demo.zipがダウンロードされるので、展開してホームに移動させる

### project

GradieとMavenの違い

> 💡GradleやMavenとは、Javaプロジェクトのビルド・依存関係管理・テストなどを自動化するための「ビルドツール」です。手作業では大変な処理を自動で行ってくれる、いわば開発の縁の下の力持ちです。<br>
> 💡GradleとMavenの違いを一言で言えば、「Gradleは柔軟で高速、Mavenはシンプルで広く使われている標準的なビルドツール」です。<br>
> 参考サイト（https://tamotech.blog/2025/06/22/intellij-spring-initializr/）

### Language

今回はJavaで開発するのでJava

### Spring Boot Version

今回選んだのは、4.0.6

Spring Boot 3系は2026年6月にサポート終了予定であるのに対し、4.0系はそれ以降もサポートが継続される<br>
参考サイト（https://spring.pleiades.io/projects/spring-boot#support）

そのため、長く使えることを考え、最新の安定版である4.0.xを選択

### Project Metadata

> Project Metadata（プロジェクトのメタデータ）:
> - Group: 組織や会社を表す識別子（例：com.mycompany）
> - Artifact: プロジェクト名（例：myproject）
> - Name: プロジェクトの表示名
> - Description: プロジェクトの簡単な説明
> - Package name: Javaのパッケージ名（通常はGroupとArtifactを組み合わせた名前）
> 参考サイト(https://dexall.co.jp/articles/?p=445)

また、別のサイトのイメージは

> Project Metadata
> - Group: 会社のドメイン名を逆にしたもの
> - Artifact: アプリ名
> - Name: アプリ名
> - Package name: なりゆきでOK
> 参考サイト（https://qiita.com/b-yuko/items/050459415dd594b704e6）

### Dependencies

- Spring Web

- Thymeleaf

- Spring Boot DevTools

この三つを追加

まず、Dependenciesとは「アプリケーションが動作するために必要な外部ライブラリ」


