# game_logの環境構築について続き

## ✅ 行ったこと

- 環境構築の解説続き

## 参考にしたサイト
https://www.kagoya.jp/howto/cloud/container/docker_laravel/

### compose.yml
game_log/compose.yml
```
services:
  app:
    container_name: app
    build: ./docker/php
    volumes:
      - .:/var/www

  nginx:
    image: nginx
    container_name: nginx
    ports:
      - 8000:80
    volumes:
      - .:/var/www
      - ./docker/nginx/default.conf:/etc/nginx/conf.d/default.conf
    working_dir: /var/www
    depends_on:
      - app

  db:
    image: mysql:5.7
    container_name: db
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: database
      MYSQL_USER: db-user
      MYSQL_PASSWORD: db-pass
      TZ: 'Asia/Tokyo'
    command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
    volumes:
      - ./docker/db/data:/var/lib/mysql
      - ./docker/db/my.cnf:/etc/mysql/conf.d/my.cnf
      - ./docker/db/sql:/docker-entrypoint-initdb.d
    ports:
      - 3306:3306
```

### `db: `
**Laravelがデータを保存するためのデータベース専用コンテナ**

### `image: mysql:5.7`
- Docker公式の MySQL 5.7 イメージを使う
- MySQLが最初からインストールされているLinuxイメージ

### `container_name: db`
- コンテナ名を db に固定
- docker exec -it db bash とか打ちやすくなる

### `environment:（MySQLの初期設定）`
**MySQLコンテナが最初に起動するときの初期設定**

- MYSQL_ROOT_PASSWORD

  MySQLの管理者(root)パスワード

- MYSQL_DATABASE

  起動時に自動作成されるデータベース名(Laravelの .env の DB_DATABASE と一致させる必要がある。)

- MYSQL_USER / MYSQL_PASSWORD

  一般ユーザーのアカウント作成(Laravelは通常 root じゃなくこのユーザーで接続する)

- TZ

  タイムゾーン設定：DBの時刻ズレ防止（日本なら必須）

### `command: ...(文字コード設定)`
**MySQLの起動コマンドを上書きし、文字コードを utf8mb4 に固定**

なぜ必要なのか？
- 絵文字・日本語の保存トラブルを避けるため
- Railsでも、`ecoding: utf8mb4`に設定したものと同じ

### `volumes:（DBデータの永続化）`
3つに分かれて役割がある<br>

①データ保存
```
- ./docker/db/data:/var/lib/mysql
```
MySQLの実データはコンテナ内 /var/lib/mysql に保存され、それを Mac側に共有している<br>
（つまり、コンテナを消してもデータは残る）<br>

②設定ファイル
```
- ./docker/db/my.cnf:/etc/mysql/conf.d/my.cnf
```
MySQLの追加設定ファイルを差し替えてる、文字コード・タイムゾーンなど細かい調整が可能<br>

③初期SQLの自動実行
```
- ./docker/db/sql:/docker-entrypoint-initdb.d
```
このフォルダに .sql ファイルを置くと、初回起動時に自動で実行される

### `ports: - 3306:3306`
Macの3306番 → DBコンテナの3306番

### `volumesの②と③の違いは、なんだろう？`
疑問？：「②はファイルを指定しているから置き換えになっていて、③はフォルダを指定しているのでファイルの追加になっている？」<br>

以下解決：

### ②のケース（ファイルをマウントしている）
何が起きているか？
```
Macのファイル
./docker/db/my.cnf
   ↓（マウント）
コンテナのファイル
/etc/mysql/conf.d/my.cnf
```
この瞬間、
- コンテナ側に元からあった my.cnf は見えなくなる
-  Mac側の my.cnf が「その場所にある唯一のファイル」になる

つまり、完全に置き換えられている<br>
コンテナの中で、
```
/etc/mysql/conf.d/my.cnf
```
を開くと、中身は 必ずMac側のファイルになっている<br>
Docker的には、「そこにあるのは“ホストのファイル”だよ」という扱いになる

### ③のケース（フォルダをマウントしている）
何が起きているか？
```
Macのフォルダ
./docker/db/sql/
   ↓（マウント）
コンテナのフォルダ
/docker-entrypoint-initdb.d/
```
この場合、
- コンテナの /docker-entrypoint-initdb.d フォルダ全体が
- Mac側の sql フォルダと同期される

コンテナにもともと存在していた、/docker-entrypoint-initdb.d フォルダの中身は見えなくなり、<br>
Mac側の ./docker/db/sql フォルダが、その場所に“差し込まれる”

## 3. 「docker」というディレクトリを作成し、その配下へ移動
```
mkdir docker
cd docker
```

## 4. 新たにphpというディレクトリを作成し、その配下へ移動
```
mkdir php
cd php
```

## 5. phpの設定ファイル「php.ini」を作成
docker/php/php.ini
```
[Date]
date.timezone = "Asia/Tokyo"
[mbstring]
mbstring.internal_encoding = "UTF-8"
mbstring.language = "Japanese"
[opcache]
opcache.memory_consumption=128
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=4000
opcache.revalidate_freq=60
opcache.fast_shutdown=1
opcache.enable_cli=1
```
ここは「Dockerよりさらに一段下のレイヤー」＝PHPそのものの設定ファイル<br>
これはコードというより：PHPの動き方を決める設定書（php.ini）といえる

### ① `[Date]` タイムゾーン設定
何をしているのか？：PHPが使う「標準の時刻」を日本時間に設定している<br>
Railsでも config.time_zone = 'Tokyo' 設定したのと同じ役割

### ② `[mbstring]` 日本語・文字コード設定
何をしているのか？：日本語（マルチバイト文字）を正しく扱えるようにする設定<br>
なぜ必要なのか？：Railsだと内部はUTF-8前提だから意識しないが、PHPは明示設定しないと事故ることがある

### ③ `[opcache]` 高速化設定
難しいが、役割はシンプル：「PHPを高速に動かすためのキャッシュ機能」

#### PHPは毎回こう動く

通常のPHPは:

- PHPファイルを読む

- 文法解析する

- 実行コードに変換する

- 実行する

これを 毎リクエストごとに繰り返す。→ 遅い

#### OPcache は何をする？

**一度変換したPHPコードをメモリに保存して再利用する**<br>

つまり、次回から解析しなくてよくなり爆速になる

#### 各設定の解説（ざっくり）

- memory_consumption

  キャッシュに使っていいメモリ量（MB）

- interned_strings_buffer

  文字列キャッシュ用メモリ

- max_accelerated_files

  キャッシュできるPHPファイルの最大数

- revalidate_freq

  ファイル変更チェック間隔（秒）<br>
  （60秒に1回だけ「ファイル変わった？」を確認する）

- fast_shutdown

  終了処理を高速化

- enable_cli

  CLI（artisan / composer）でもOPcacheを有効化

### まとめ：php.ini
- Date：時効ズレ防止
- mbstring：日本語文字化け防止
- opcashe：PHP高速化

つまり、Laravelが安定・高速・日本語安全に動くための土台設定
