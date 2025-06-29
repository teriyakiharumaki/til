# credentialsって何？

## ✅ 行ったこと

- credentialsについて調べた

## credentialsとは

- 一言でいうと：アプリケーションの秘密情報（APIキー、パスワードなど）を安全に管理するための仕組みです。[GPT]
- 詳しく説明すると：Railsには、秘密情報（例：APIキーやデータベースパスワードなど）をソースコードに直接書かずに、安全に保管する方法が用意されています。それがcredentials.yml.encという暗号化ファイルです。[GPT]<br>
  - ファイルの実体は：config/credentials.yml.enc（暗号化された状態）<br>
  - このファイルを編集・復号化するために必要なのが：config/master.key（暗号化・復号化に使う鍵）
- Rails の credentials は、アプリケーションで使用する機密情報や設定情報を安全に管理する仕組み。[1]
- 主に API キー、シークレットキー、パスワードなどを格納するために使われる。[1]
- これらの情報は暗号化され、config/credentials.yml.enc ファイルに保存される。[1]

## つまりcredentialsとは
credentialsは、Railsアプリケーションで使用する機密情報や設定情報を安全に管理するための仕組みです。
APIキーなどの重要な情報を暗号化されたファイルに保存することで、情報の漏洩を防ぐことができます。
復号には master.key が必要で、セキュリティの高い管理が可能となっています。

## 参考サイト
[1](https://keita1899.hatenablog.com/entry/2025/02/10/132816#:~:text=Rails%20%E3%81%AE%20credentials%20%E3%81%AF%E3%80%81%E3%82%A2%E3%83%97%E3%83%AA%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3,%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%81%AB%E4%BF%9D%E5%AD%98%E3%81%95%E3%82%8C%E3%82%8B%E3%80%82)<br>
[【秘匿情報の管理】「credentials.yml.encとmaster.key」「.env」の使い方とメリットを比較](https://zenn.dev/kame0707/articles/ef2453f31fe236)
