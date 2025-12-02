# つぶやきができるサイト「Tubuyaki」を作る

## ✅ 行ったこと

- 環境構築を行なった

## ディレクトリを作成し、移動
```
mkdir tubuyaki
cd tubuyaki
```

## Docker系ファイルの作成
Dockerfile.devファイルを生成
```
touch Dockerfile.dev
```

compose.ymlファイルを生成
```
touch compose.yml     
```

## Dockerfile.devの設定
```
# Ruby のベースイメージを指定
FROM ruby:3.3.9

# 環境変数の設定
ENV LANG C.UTF-8  # 日本語対応
ENV TZ Asia/Tokyo # タイムゾーンを日本に設定

# 必要なパッケージのインストール
RUN apt-get update -qq \
&& apt-get install -y ca-certificates curl gnupg \
&& mkdir -p /etc/apt/keyrings \
&& curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg \
&& NODE_MAJOR=20 \
&& echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list \
&& wget --quiet -O - /tmp/pubkey.gpg https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
&& echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs yarn vim

# アプリケーションディレクトリの作成
RUN mkdir /myapp
WORKDIR /myapp

# Bundlerのインストール
RUN gem install bundler

# アプリケーションのコードをコピー
COPY . /myapp
```

## compose.yml を編集し、PostgreSQLを使用する設定
```
services: # コンテナの定義開始
  db: # PostgreSQLデータベースの設定
    image: postgres # PostgreSQLの公式イメージを使用
    restart: always # コンテナを常に再起動
    environment: # 環境変数の設定
      TZ: Asia/Tokyo # タイムゾーンを日本に設定
      POSTGRES_PASSWORD: password # データベースのパスワード設定
    volumes: # データの永続化設定
      - postgresql_data:/var/lib/postgresql # DBデータを永続化
    ports: # ポート設定
      - 5432:5432
    healthcheck: # コンテナの健康チェック設定
      test: ["CMD-SHELL", "pg_isready -d myapp_development -U postgres"]
      interval: 10s # チェック間隔
      timeout: 5s # タイムアウト時間
      retries: 5 # リトライ回数
      
  web: # Railsアプリケーションの設定
    build: # Dockerイメージのビルド設定
      context: .  # ビルドコンテキストはカレントディレクトリ
      dockerfile: Dockerfile.dev # 開発用Dockerfileを使用
    command: bash -c "bundle install && bundle exec rails db:prepare && rm -f tmp/pids/server.pid && ./bin/dev"
    # ↑ サーバー起動時のコマンド：
    # - bundle install: 必要なgemをインストール
    # - rails db:prepare: DBの準備
    # - サーバープロセスIDファイルの削除
    # - 開発サーバーの起動

    tty: true # 疑似ターミナルを割り当て
    stdin_open: true # 標準入力を保持
    volumes:  # ボリュームマウントの設定
      - .:/myapp # ソースコードのマウント
      - bundle_data:/usr/local/bundle:cached # gemの永続化
      - node_modules:/myapp/node_modules # node_modulesの永続化
    environment: # 環境変数の設定
      TZ: Asia/Tokyo # タイムゾーンを日本に設定
    ports: # ポート設定
      - "3000:3000" # Rails標準のポート
    depends_on: # 依存関係の設定
      db: # dbサービスに依存
        condition: service_healthy # dbの健康チェックが通るまで待機

volumes: # ボリュームの定義
  bundle_data: # gemデータの永続化用ボリューム
  postgresql_data: # PostgreSQLデータの永続化用ボリューム
  node_modules: # Node.jsパッケージの永続化用ボリューム
```

### compose build(Bootstrapもあり)
```
docker compose build
docker compose run --rm web gem install rails
docker compose run --rm web rails new . -d postgresql -j esbuild --css=bootstrap
```

### エラー発生！
```
ERROR [web 2/7] RUN apt-get update -qq && apt-get install -y ca-certi  4.3s
```
・apt-key コマンドが 削除された（非推奨）ため使えなくなっている
・Debian Bookworm（ruby:3.3.9-bookworm など）以降では apt-key は完全に無効化されている

### 🎯 解決策：apt-key を使わない形にDockerfileを書き換える
```
# Node.js & Yarnのインストール（apt-key不使用版）
RUN curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /usr/share/keyrings/nodesource.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/nodesource.gpg] https://deb.nodesource.com/node_20.x nodistro main" > /etc/apt/sources.list.d/nodesource.list && \
    curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor -o /usr/share/keyrings/yarn.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/yarn.gpg] https://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list && \
    apt-get update -qq && apt-get install -y nodejs yarn
```

修正後
```
# Ruby のベースイメージを指定
FROM ruby:3.3.9

# 環境変数の設定
ENV LANG C.UTF-8  # 日本語対応
ENV TZ Asia/Tokyo # タイムゾーンを日本に設定

# 基本パッケージのインストール
RUN apt-get update -qq && \
    apt-get install -y --no-install-recommends \
    build-essential \
    libpq-dev \
    curl \
    gnupg \
    ca-certificates

# Node.js & Yarn の設定（apt-keyを使わない）
RUN curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /usr/share/keyrings/nodesource.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/nodesource.gpg] https://deb.nodesource.com/node_20.x nodistro main" > /etc/apt/sources.list.d/nodesource.list && \
    curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor -o /usr/share/keyrings/yarn.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/yarn.gpg] https://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list && \
    apt-get update -qq && apt-get install -y nodejs yarn

# アプリケーションディレクトリの作成
RUN mkdir /myapp
WORKDIR /myapp

# Bundlerのインストール
RUN gem install bundler

# アプリケーションのコードをコピー
COPY . /myapp
```

## docker compose up 実行
実行したが、ページが表示されず
```
tubuyaki-db-1   | 2025-08-16 16:22:25.617 JST [1188] FATAL:  database "myapp_development" does not exist
```
これは、
- docker compose up でRailsが db:prepare を自動実行
- でも db:create していないので myapp_development というDBがない
- そのため「PostgreSQLにつながらないよ！」というエラーが出ているだけ

以下を実行
```
docker compose run --rm web bin/rails db:create
```

###　さらなるエラー
```
connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: No such file or directory
```
Railsコンテナ（web）が、PostgreSQLに接続しようとしたけど見つからなかった<br>
PostgreSQLコンテナ（db）は 起動していないか、設定が足りない可能性が高い<br>

### 解決策
原因：config/database.yml の host: 設定が空 or localhost になっている<br>
対応：config/database.yml の development: に 明示的に host: db を追加
```
development:
  <<: *default
  database: tubuyaki_development
  host: db   
```
### なぜ host: db が必要なのか？
docker-compose.yml で定義している db: サービスにRailsが接続するには、<br>
host: db と明記しないとRailsは「ローカルのPostgreSQL」に接続しようとしてしまう。<br>
デフォルトのままだと /tmp/.s.PGSQL.5432 のようなUnixソケットに接続を試みる → 失敗。

### さらなるエラー
```
PG::ConnectionBad: connection to server at "172.20.0.2", port 5432 failed: fe_sendauth: no password supplied
```
→PostgreSQLコンテナ側が パスワード認証を要求しているが、Rails の database.yml に password: の設定がない<br>

### 解決策
docker-compose.yml に環境変数でパスワード設定 → Rails側も合わせる<br>
docker-compose.yml の db: に環境変数を追加：
```
services:
  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: passward
```
これはもともと設定していた<br>
config/database.yml の development に追記：
```
development:
  <<: *default
  database: tubuyaki_development
  host: db
  username: postgres
  password: passward # ←ここを追加！
```

### 2025/12/3追記
```
default: &default
  adapter: postgresql
  encoding: unicode
  # For details on connection pooling, see Rails configuration guide
  # https://guides.rubyonrails.org/configuring.html#database-pooling
  max_connections: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  host: db
  username: postgres
  password: password
```
testにもusernameとpasswardを追記しないと同様のエラーが出るので、defaultに記述するべき（developmentとtestからは削除）

### さらなるエラー
サーバーは起動しているように見えるが、「localhost からデータが送信されませんでした。」と表示されサイトが表示されない