# DBの作成

## ✅ 行ったこと

- DBの作成を行った
- マイグレーション時に起きたエラーの解決を行った

## コンテナ内のartisanがある場所まで移動し、gameモデルの作成
```
root@020990d08807:/var/www# cd game_log
root@020990d08807:/var/www/game_log# php artisan make:model Game -m

Deprecated: PHP Startup: Use of mbstring.internal_encoding is deprecated in Unknown on line 0

   INFO  Model [app/Models/Game.php] created successfully.  

   INFO  Migration [database/migrations/2026_02_06_165045_create_games_table.php] created successfully.  
```
game_log/app/Models/Game.phpと、game_log/database/migrations/2026_02_06_165045_create_games_table.php
が作成された

## DBのカラム一覧
- title（ゲーム名）

- platform

- rating（★1〜5）

- review（感想）

- price（値段）

- condition（新品 or 中古）

- status（未プレイ / プレイ中 / クリア済み）

  まずはシンプルに string で実装（status : unplayed / playing / cleared）

- play_time_minutes（プレイ時間｛分｝）

  プレイ時間は集計しやすいように 分（minutes）を整数で保存が扱いやすい

- purchased_at（購入日）

- cleared_at（クリア日）

## マイグレーションファイルの中身を追記
game_log/database/migrations/2026_02_06_165045_create_games_table.php
```
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('games', function (Blueprint $table) {
            $table->id();
            $table->string('title');
            $table->string('platform')->nullable();
            $table->integer('rating')->nullable();
            $table->text('review')->nullable();
            $table->integer('price')->nullable();
            $table->string('condition')->nullable();
            $table->string('status')->default('unplayed');
            $table->integer('play_time_minutes')->nullable();
            $table->date('purchased_at')->nullable();
            $table->date('cleared_at')->nullable();
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('games');
    }
};

```

`Schema::create('games', function (Blueprint $table) {`内を追記

そして、マイグレーションを実行
```
root@020990d08807:/var/www/game_log# php artisan migrate
```

マイグレーション参考サイト<br>
(https://qiita.com/Takahiro_Nago/items/71d30873313862ab6818)

## エラー発生❗️❗️❗️
Laravelのmigration自体は正しいんだけど、DBに繋がってないエラーが発生
```
Deprecated: PHP Startup: Use of mbstring.internal_encoding is deprecated in Unknown on line 0

   Illuminate\Database\QueryException 

  SQLSTATE[HY000] [2002] Connection refused (Connection: mysql, SQL: select table_name as `name`, (data_length + index_length) as `size`, table_comment as `comment`, engine as `engine`, table_collation as `collation` from information_schema.tables where table_schema = 'laravel' and table_type in ('BASE TABLE', 'SYSTEM VERSIONED') order by table_name)

  at vendor/laravel/framework/src/Illuminate/Database/Connection.php:829
    825▕                     $this->getName(), $query, $this->prepareBindings($bindings), $e
    826▕                 );
    827▕             }
    828▕ 
  ➜ 829▕             throw new QueryException(
    830▕                 $this->getName(), $query, $this->prepareBindings($bindings), $e
    831▕             );
    832▕         }
    833▕     }

      +39 vendor frames 

  40  artisan:35
      Illuminate\Foundation\Console\Kernel::handle(Object(Symfony\Component\Console\Input\ArgvInput), Object(Symfony\Component\Console\Output\ConsoleOutput))
```
**Laravelのmigration自体は正しいんだけど、DBに繋がってないエラー**

ポイント
```
SQLSTATE[HY000] [2002] Connection refused (Connection: mysql)
```
**= MySQLに接続しようとしたけど拒否された（繋がらない）**
- MySQLコンテナが起動してない / 落ちてる
- .env の DB_HOST が間違ってる（127.0.0.1 になってる等）
- サービス名（ホスト名）が違う（docker-composeで db なのに mysql になってる等）

***

## 直す手順

### ①コンテナの起動確認（ホスト側のターミナル）
root@... じゃない方（Mac側）で、game_log ディレクトリ：
```
docker compose ps
```
を行う

```
〜MacBook-Pro game_log % docker compose ps
NAME                IMAGE               COMMAND                  SERVICE             CREATED             STATUS              PORTS
app                 game_log-app        "docker-php-entrypoi…"   app                 5 minutes ago       Up 5 minutes        9000/tcp
db                  mysql:5.7           "docker-entrypoint.s…"   db                  5 minutes ago       Up 5 minutes        0.0.0.0:3306->3306/tcp, 33060/tcp
nginx               nginx               "/docker-entrypoint.…"   nginx               5 minutes ago       Up 5 minutes        0.0.0.0:8000->80/tcp

```

dbはUPとなっており、起動はできている

***

### ②.env の DB設定を確認（Laravel側）
コンテナ内で game_log/.env を確認：
```
root@020990d08807:/var/www/game_log# grep -E "DB_CONNECTION|DB_HOST|DB_PORT|DB_DATABASE|DB_USERNAME|DB_PASSWORD" .env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=

```
これの内容とcompose.ymlの中身を確認し、違いがないか調べる
compose.yml
```
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
原因：.env が ローカルPC用の設定のままになってる
- DB_HOST=127.0.0.1 ← コンテナ内の「自分自身」を指すので dbコンテナに繋がらない
- DB名/ユーザー/パスも compose.yml とズレてる

なので、以下のように.envを修正
```
DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=database
DB_USERNAME=db-user
DB_PASSWORD=db-pass
```

#### .envを検索するコードについて
```
grep -E "DB_CONNECTION|DB_HOST|DB_PORT|DB_DATABASE|DB_USERNAME|DB_PASSWORD" .env
```
1. grep -Eについて

  （https://eng-entrance.com/linux-command-grep#-E）

2. "DB_CONNECTION|DB_HOST|..."

  "|"(ハイプ)を利用してOR検索をしている<br>
  （https://qiita.com/P-man_Brown/items/575c1aaad0e577c02aad）

3. 検索対象ファイルは、.env(1と同じ参考サイト)

***

### ③Laravelは設定をキャッシュしてることがあるから、クリアする
```
root@020990d08807:/var/www/game_log# php artisan config:clear

Deprecated: PHP Startup: Use of mbstring.internal_encoding is deprecated in Unknown on line 0

   INFO  Configuration cache cleared successfully.  

root@020990d08807:/var/www/game_log# php artisan cache:clear

Deprecated: PHP Startup: Use of mbstring.internal_encoding is deprecated in Unknown on line 0

   INFO  Application cache cleared successfully.  
```

***

## マイグレーション
```
root@020990d08807:/var/www/game_log# php artisan migrate

Deprecated: PHP Startup: Use of mbstring.internal_encoding is deprecated in Unknown on line 0

   INFO  Preparing database.  

  Creating migration table ................................................................................ 37ms DONE

   INFO  Running migrations.  

  2014_10_12_000000_create_users_table .................................................................... 38ms DONE
  2014_10_12_100000_create_password_reset_tokens_table ..................................................... 7ms DONE
  2019_08_19_000000_create_failed_jobs_table .............................................................. 18ms DONE
  2019_12_14_000001_create_personal_access_tokens_table ................................................... 22ms DONE
  2026_02_06_165045_create_games_table ..................................................................... 8ms DONE
```
