# docker-compose up

## ✅ 行ったこと

- docker-compose upについて調べた

## docker-compose upとは

- docker-composeとは

  Dockerという、様々なソフトウェアを動作環境ごとパッケージングして、どんなパソコンでも手軽に動かせるようにした仕組みがあります。<br>

  docker-composeはそのDockerのパッケージを複数組み合わせて動かす時に使うものです。<br>

  ちなみにDockerのパッケージの事をイメージと呼びます。

- docker-compose upが何してるのさ

  docker-compose upはdocker-composeを使ってイメージを元にソフトウェアを動かす際に用いるコマンドです。雑に言えば起動コマンドです。<br>

  イメージを元に起動した物をコンテナと呼びます。イメージとコンテナの関係は金型と生成物の関係に似ていますね。

- 地獄からの使者、docker-compose.yml

  docker-composeがどうやって起動するイメージを把握するのか。答えはdocker-composeを使うディレクトリにあります。<br>

  docker-compose upを使うと、使ったディレクトリにあるdocker-compose.ymlというファイルと、docker-compose.override.ymlという二つのファイルを(存在していれば)自動で読み取ります。<br>

  これらのファイルにどんなコンテナを動かせばいいのかが書いてあるという訳です。<br>

  優秀な点として、docker-compose.yml達にイメージの名前を書いておけば、例えイメージが手元になくても、イメージが必要になった段階で自動でダウンロートして来てくれます。

## 参考サイト
[1](https://qiita.com/negineri/items/8fe51fa116a16a371250)<br>
