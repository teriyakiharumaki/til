# game_logの環境構築について続き

## ✅ 行ったこと

- 環境構築の解説続き

## 参考にしたサイト
https://www.kagoya.jp/howto/cloud/container/docker_laravel/

## 6. Dockerコンテナを起動するのに使う「Dockerfile」を作成
docker/php/Dockerfile
```
FROM php:8.2-fpm
COPY php.ini /usr/local/etc/php/

RUN apt-get update \
  && apt-get install -y zlib1g-dev mariadb-client vim libzip-dev \
  && docker-php-ext-install zip pdo_mysql

#Composer install
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
RUN php composer-setup.php
RUN php -r "unlink('composer-setup.php');"
RUN mv composer.phar /usr/local/bin/composer

ENV COMPOSER_ALLOW_SUPERUSER 1

ENV COMPOSER_HOME /composer

ENV PATH $PATH:/composer/vendor/bin


WORKDIR /var/www

RUN composer global require "laravel/installer"
```

### Dockerfileがやってることの全体像
- ベース：php:8.2-fpm（PHP-FPM入り）
- PHP設定：php.ini を適用
- 必要なLinuxパッケージを入れる
- PHP拡張を入れる（zip / pdo_mysql）
- Composerを入れる
- Laravel installer を入れる
- 作業場所を /var/www にする

Laravelが動く「作業場」をコンテナ内に作ってるイメージ

### ①ベースイメージの土台
```
FROM php:8.2-fpm
```
- 「PHP 8.2 と PHP-FPM が入ってる公式イメージ」を土台にする
- fpm は 「FastCGI Process Manager」 の略で、nginxと組み合わせてPHPを実行する方式

イメージ：
  - nginx：受付
  - php-fpm：PHP実行エンジン（Laravelを動かす）

### ②php.ini をコンテナに入れる
```
COPY php.ini /usr/local/etc/php/
```
作った docker/php/php.ini をコンテナ内の所定場所にコピーしている（この場所は、公式PHPイメージが php.ini を読み込む標準ディレクトリ）<br>
→これで timezone / mbstring / opcache などがコンテナ内PHPに反映される

### ③Linuxパッケージのインストール + PHP拡張
```
RUN apt-get update \
  && apt-get install -y zlib1g-dev mariadb-client vim libzip-dev \
  && docker-php-ext-install zip pdo_mysql
```

#### `apt-get update`
Debian系Linuxの「パッケージ一覧」を更新

#### `apt-get install -y ...`
必要な道具のインストール
- zlib1g-dev：圧縮系（zip拡張ビルドに使うことが多い）
- libzip-dev：zip拡張のためのライブラリ
- mariadb-client：MySQL/MariaDBに接続するためのクライアント（mysql コマンドなど）
- vim：コンテナ内で編集したい時用（好み）

#### `docker-php-ext-install zip pdo_mysql`
PHP拡張を有効化してビルド・インストールする公式のヘルパー
- zip：Zipファイルを扱えるようにする
- pdo_mysql：LaravelがMySQLに接続するためにほぼ必須（これがないと .env のDB接続でエラーになる）

### ④Composerのインストール
```
#Composer install
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
RUN php composer-setup.php
RUN php -r "unlink('composer-setup.php');"
RUN mv composer.phar /usr/local/bin/composer
```
ComposerはPHPの「gem」みたいなもの

#### 具体的に行われていること
- インストーラーをダウンロード
- 実行して composer.phar を作る
- インストーラーファイルを削除
- composer コマンドとして使える場所に移動

結果、コンテナ内で
```
composer --version
```
が使えるようになる

### ⑤Composerの環境変数
```
ENV COMPOSER_ALLOW_SUPERUSER 1
```
- コンテナ内はrootユーザーで作業することが多い
- Composerはroot実行を嫌がるので、その警告/制限を緩める設定

```
ENV COMPOSER_HOME /composer
```
Composerのホームディレクトリ（設定やキャッシュ置き場）を /composer にする

```
ENV PATH $PATH:/composer/vendor/bin
```
- Composerのグローバルインストールしたコマンド（vendor/bin）をPATHに通す
- これにより laravel コマンドなどが打てるようになる

### ⑥作業ディレクトリ
```
WORKDIR /var/www
```
- 以降の RUN は /var/www で実行される
- docker compose exec app bash で入ったときも、ここが基準になることが多い

（compose.yml の volumes: .:/var/www と揃ってるのが良い）

### ⑦Laravel installer をグローバルに入れる
```
RUN composer global require "laravel/installer"
```
- laravel new xxx で新規プロジェクト作成できるやつ
- ただし最近は、composer create-project laravel/laravel ... で作る人も多いので必須ではない

### Dockerfileでやってることまとめ
- php:8.2-fpm：nginxと組むためのPHP実行環境
- pdo_mysql：MySQL接続に必須
- php.ini：タイムゾーン・文字コード等を安定化
- composer：Laravel依存管理＆プロジェクト作成に必須
- laravel/installer：Laravelコマンドで雛形生成できるように

## 7. phpディレクトリから１つ上のディレクトリ（docker）に移動し、新たなディレクトリ「nginx」を作成して移動する
```
cd ..
mkdir ngnix
cd ngnix
```

## 8. Webサーバー「nginx」の設定ファイル「default.conf」を作成
docker/nginx/default.conf
```
server {
  listen 80;
  root /var/www/game_log/public;
  index index.php;

  location / {
    root /var/www/game_log/public;
    index index.php;
    try_files $uri $uri/ /index.php$query_string;
  }

  location ~ .php$ {
    try_files $uri =404;
    fastcgi_split_path_info ^(.+.php)(/.+)$;
    fastcgi_pass app:9000;
    fastcgi_index index.php;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param PATH_INFO $fastcgi_path_info;
  }
}
```
参考サイトと違い、環境構築用ディレクトリを「Larabel_docker」ではなく、「game_log」にしたため、サイトのコードを変更（***）

### default.conf がやっていること
```
ブラウザからアクセスが来る
   ↓
nginx が受け取る
   ↓
・CSS / 画像 → nginx がそのまま返す
・PHP（Laravel） → app(PHP-FPM) に渡す
```
***「静的ファイルは自分で返す」「PHPは app コンテナに投げる」*** <br>
というルールを書いている

### ①`server { ... }`
```
server {
  ...
}
```
nginxの「1つのWebサイト設定」

### ②`listen 80;`
nginxが 80番ポートで待ち受ける<br>

Docker Compose で：
```
ports:
  - 8000:80
```
で書いていたので、
```
ブラウザ → localhost:8000 → nginx:80
```
になる

### ③`root /var/www/game_log/public;`
***Webサイトのルートディレクトリ*** <br>

ブラウザから / にアクセスされたら、nginx は /var/www/game_log/public を見に行く<br>
Laravelでは、
```
public/
  ├ index.php
  ├ css/
  ├ js/
```
このpublicだけを公開する

### ④`index index.php;`
/ にアクセスされたとき、自動的に index.php を探す<br>
（Railsのルーティングに近い感覚）

### ⑤`location / { ... }（通常アクセス）`
```
location / {
  root /var/www/game_log/public;
  index index.php;
  try_files $uri $uri/ /index.php$query_string;
}
```
***「通常のURLアクセス」をどう処理するか***

#### `try_files`が大事
```
try_files $uri $uri/ /index.php$query_string;
```
意味は：
  - $uri（実ファイル）があればそれを返す
  - $uri/（ディレクトリ）があればそれを返す
  - どちらも無ければ /index.php に渡す

#### なぜこれが必要？
Laravelは：
```
/games
/reviews/1
/login
```
みたいなURL全部を index.php → Laravelルーターで処理する<br>
でもnginxは本来：<br>
  「そのファイル存在するのか？」しか見ない<br>
なので、
```
/games → ファイル無い → 404
```
になってしまう<br>
それを、「無ければ全部index.phpに投げろ」
と指示しているのがtry_files

### ⑥`location ~ .php$ { ... }（PHP処理）`
```
location ~ .php$ {
  ...
}
```
.php で終わるリクエストだけを処理するブロック<br>
たとえば、
```
/index.php
/some.php
```
だけここに入る

#### 🚚 ここから「PHPを app コンテナに渡す」設定

- `fastcgi_pass app:9000;`

  ***nginx→appコンテナの9000番ポートに処理投げる***<br>
  appはsompose.ymlのサービス名<br>
  Dockerネットワーク内では
  ```
  app:9000 = PHP-FPM
  ```
  として通信できる

- `include fastcgi_params;`

  ***PHPに渡すための標準のパラメータセット***

- `fastcgi_param SCRIPT_FILENAME ...`

  ***実行するPHPファイルのフルパスをPHPに伝える***<br>
  例：
  ```
  /var/www/game_log/public/index.php
  ```

- `fastcgi_param PATH_INFO ...`

  ***URLのパス情報をPHPに渡す***
  Laravelルーティングに必要

- その他の処理

  ```
  try_files $uri =404;
  fastcgi_split_path_info ^(.+.php)(/.+)$;
  fastcgi_index index.php;
  ```
  - 存在しないPHPファイルは404にする
  - パス情報を正しく分解する
  - index.php を基準にする

## 9. game_log直下に移動して、compose.ymlを使い、Dockerコンテナを起動する
```
cd ..
cd ..
docker compose up -d
```

# Laravelアプリを作成

## 1. Dockerコンテナの中に入る
```
docker compose exec app bash
```

## 2. コンテナ内で以下のコマンドを実行
```
composer create-project --prefer-dist laravel/laravel laravel-project "10.*"
```
***Laravelのひな形をダウンロードして、新しいアプリを作るコマンド***<br>
Railsで言う
```
rails new app
```
とほぼ同じ

### ①composer
***PHPのパッケージマネージャ***

（Rubyでいう：bundler / gem, Nodeでいう：npm）<br>

Laravel本体も Composer で管理されている

### ②create-project
***「テンプレートから新しいプロジェクトを作る」***<br>

Composerのサブコマンド

- GitHubから laravel/laravel テンプレを取得
- 必要な依存ライブラリを自動インストール
- 初期ディレクトリ構成を作成

を一気にやる

### ③--prefer-dist
***「できるだけ zip などの配布パッケージでダウンロードする」***

- Git clone より高速
- 安定した配布物を使う
- 開発用ソースじゃなくリリース版を取る

### ④laravel/laravel
***「どのテンプレートを使うか」***

- Composer上のパッケージ名
- Laravel公式のアプリ雛形テンプレ

### ⑤laravel-project
***「作成されるファイル名」***

この名前のディレクトリが作られて：
```
laravel-project/
  ├ app/
  ├ routes/
  ├ composer.json
  ├ artisan
```
が生成される

### ⑥"10.*"
***Laravelのバージョン指定」***

"10.*" = Laravel 10系の最新安定版<br>
Railsでいう
```
gem 'rails', '~> 7.1'
```
みたいな指定

### ⚠️フォルダ名変更
今回はプロジェクト名を「game_log」で進めていたので、コンテナ内で名前を変更
```
mv laravel-project game_log
```

## 実際に行われている処理
このコマンドを実行すると：

  1. Composerが laravel/laravel を探す

  2. 指定バージョン（10系）を取得

  3. 新しいフォルダ laravel-project を作成

  4. 必要なライブラリを全部ダウンロード

  5. .env や初期設定ファイルを生成

  6. autoload設定を作る

→ すぐに動くLaravelアプリが完成

## 3. storageディレクトリの所有者をwww-dataに変更
```
cd game_log
chown www-data storage/ -R
```

### ①cd game_log
game_logのディレクトリに移動している

### ②chown www-data storage/ -R
***storage フォルダの「持ち主」を www-data に変更するという意味***

#### chown って何？
chown = change owner（所有者を変える）<br>
Linuxでは、すべてのファイルに：
- 👤 所有者（誰のものか）
- 👥 グループ
- ✍️ 権限（読み・書き・実行）

が設定されている

#### www-data って誰？
www-data は：***Webサーバ（nginx / PHP）が使う専用ユーザー***

多くのLinux環境で：
  - nginx
  - Apache
  - PHP-FPM

は www-data ユーザーとして動く（つまり： ***webアプリ本人のユーザー*** みたいな存在）

### なぜstorageの所有者を変えるのか
Laravelでは：
```
storage/
  ├ logs/
  ├ framework/cache/
  ├ framework/sessions/
  ├ uploads/
```

などに、アプリが 実行中にファイルを書き込む

例えば：

- ログファイルを書く
- キャッシュを書く
- セッション保存する
- 画像アップロードする

***もし所有者が違うと…***

仮に storage の所有者が：
```
root
```

だった場合：

- PHP（www-data）が書き込みしようとする
- ❌ 権限がない
- ❌ Permission denied エラー
- ❌ アプリが落ちる

という事故が起きる

***だからこうする***
```
chown www-data storage/ -R
```

にすることによって：

- ✔ storage の中身は全部
- ✔ Webアプリ本人（www-data）が自由に触れる

状態にする

### ③-Rの意味
中のフォルダ・ファイル全部を意味する

### 所有者変更のまとめ
***Laravelが実行中にファイルを書けるように、storageの持ち主をWebアプリに変えている***