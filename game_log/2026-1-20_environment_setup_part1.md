# ゲームのレビューができるができるサイト「game_log」を作る

## ✅ 行ったこと

- 環境構築を行なった

## 参考にしたサイト
https://www.kagoya.jp/howto/cloud/container/docker_laravel/

## 環境構築の流れ

## 1. ディレクトリを作成し、移動
```
mkdir game_log
cd game_log
```

## 2. compose.ymlを作成
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

### `services:`
- ここから下に、コンテナたちの定義を並べる
- app, nginx, dbは別のコンテナ

### `app:(PHPコンテナの名前＝サービス名）`
- app は Laravel（PHP）を実行するコンテナ
- 役割：
  - php artisan ...
  - composer install
  - Laravelのコード実行（PHP-FPM）みたいな “アプリ本体” になる

  ※ ブラウザから直接アクセスするのは nginx で、app は裏側でPHPを処理する

### `container_name: app`
- Docker Composeは通常、コンテナ名を自動生成する（例：game_log-app-1）
- でもこれを書くと、コンテナ名が強制的に app になる

**メリット：** docker logs app とか docker exec -it app bash みたいに短く打てる

### `build: ./docker/php`
image: じゃなくて build: を使ってるので、これは **自分でDockerイメージを作る** という指定<br>

./docker/php ディレクトリの中にある Dockerfile を使って、PHP環境（php-fpm、composer、拡張モジュール etc）を入れた **自作のPHP用イメージ** をビルドする

### `volumes: - .:/var/www`
ホスト（Mac）の game_log ディレクトリ . を、コンテナ内の /var/www に **マウント（共有）** している

- MacでVSCodeで編集したファイルが、すぐにコンテナ内にも反映される
- 逆に、コンテナ内で composer create-project して作ったLaravelも、Mac側のフォルダにちゃんと生成される

### `appのまとめ`
- app = Laravel(PHP)を動かすコンテナ
- build で PHP環境を自作
- volumes で Macのコードとコンテナを同期
- container_name は 操作を楽にする（ただし衝突注意する）

### `nginxとは`
**ブラウザからのアクセスを最初に受け取る「受付係」**

#### Railsとの対比

  **Railsでは、**
  ```
  ブラウザ → Puma（Rails） → アプリ処理 → HTML返す
  ```
  となっており、Pumaが以下を全て実行している
  - HTTPリクエストを受ける
  - ルーティングも処理する
  - 静的ファイルも返す
  - Rubyコードを実行する

  **Laravel + nginx + PHP-FPMでは、**
  ```
  ブラウザ → nginx（受付・振り分け） → PHP-FPM（PHP実行） → Laravel（アプリ処理）
  ```
  **nginxの役割**
  - HTTPリクエストを受け取る
  - 画像・CSS・JSなどの静的ファイルを返す
  - PHPリクエストをPHP-FPMに渡す
  - 高速で軽い

  nginx自身は PHPもLaravelも一切知らない<br>
    「このURL来たから、このファイル見せる」<br>
    「このリクエストはPHPに渡す」<br>
  という役割<br>

### `nginx:（サービス名）`
このサービスは「nginxコンテナ」で、Webサーバ担当

### `image: nginx`
Docker Hubの公式 nginx イメージを使う<br>
中身は：
- nginx がインストール済み
- 最低限のLinux環境

Laravelは入ってない

### `container_name: nginx`
- コンテナ名を固定で nginx にする
- docker logs nginx とかしやすい

### `ports: - 8000:80`
「Macの8000番ポート → nginxコンテナの80番ポート」というポート転送を行う

**実際の流れ**
ブラウザで：
```
http://localhost:8000
```
にアクセスすると、
1. Macの8000番にリクエストが来る
2. Dockerがそれを nginxコンテナの80番に転送
3. nginxがリクエストを受け取る

### `volumes:`
よくわからなかったので、そもそもから調べる

# 前提として、コンテナは別の小さいPC
Dockerコンテナは、**🖥️ Macとは完全に別のミニPC** <br>
本来なら：
- Macのファイル → コンテナから見えない
- コンテナのファイル → Macから見えない
完全に分離されてる

## ここで登場するのがvolumes(共有フォルダ)
volumes は一言でいうと、**📁 Macとコンテナでフォルダを共有する設定**

### `- .:/var/www`
意味：
```
Macの「今いるフォルダ(.)」
        ↓ 共有
コンテナの「/var/www」
```

#### 何が起きてるのか？
現在いるフォルダ
```
game_log/
  ├ docker/
  ├ compose.yml
  ├ game_log/（Laravel）
```
. は「この game_log フォルダ」の意味<br>
それを、コンテナの /var/wwwにコピーじゃなくて、**リアルタイム共有**

#### できること
- Macでファイル編集すると
- コンテナ側にも即反映される
- nginx も PHP も同じコードを見て動く

もしこれが無かったら：😱 Macでコードを書いても、コンテナは別世界なので何も見えない

### `- ./docker/nginx/default.conf:/etc/nginx/conf.d/default.conf`
意味：
```
Macの
./docker/nginx/default.conf
        ↓ 共有
コンテナの
/etc/nginx/conf.d/default.conf
```

#### 何をしているのか
nginx は通常：
```
/etc/nginx/conf.d/default.conf
```
という場所の設定ファイルを読み込む<br>
でもそのファイルを、「Mac側で自分が編集したファイルに差し替えてる」という設定<br>

**なぜやるのか：nginx の設定を、コンテナの中に入らなくてもMacのエディタで編集したいから**

### `working_dir: /var/www`
意味：「コンテナに入ったときの最初の場所」<br>

普通では、
```
docker compose exec nginx bash
```
でコンテナに入ると、/から始まる
しかし、この設定があると
```
/var/www
```
からスタートする、Laravelのコードがある場所だから便利

### `depends_on: - app`
意味：nginx は app(PHP) より後に起動する<br>
なぜ必要？：nginx は、リクエストを PHP(app) に投げる役割がある<br>
もし
- nginx が先に起動
- app がまだ起動してない

と、一瞬エラーになる可能性がある、それを避けるための「起動順ルール」

### nginxの全体の動き
```
Mac（game_log）
 ├ コード
 ├ default.conf
 └ compose.yml
      ↑
      │ volumesで共有
      ↓
コンテナ（nginx）
 ├ /var/www  ← Laravelコードが見える
 └ /etc/nginx/conf.d/default.conf ← 設定を差し替え
```
nginxは：

- ブラウザからアクセスを受け取り
- /var/www/public を見に行き
- PHPは app に投げる

という動きをしている