# 環境構築時のエラーについて

## ✅ 行ったこと

- サーバーが立ち上がるのに、localhost:3000に接続してもサイトが表示されないエラーについて調べた

## 解決
Procfile.dev を編集していなかった

```
web: env RUBY_DEBUG_OPEN=true bin/rails server -b 0.0.0.0 -p 3000 #-b 0.0.0.0 -p 3000を追記
js: yarn build --watch
css: yarn watch:css
```

## なぜこの記述でブラウザからアクセスできるようになるのか

### 💡 -b 0.0.0.0 の意味（bind address）
Railsの開発サーバー（Puma）は、デフォルトでは localhost（=127.0.0.1）でしか待ち受けていない<br>
でも、Dockerコンテナの中で localhost を使うと、「コンテナの中だけの localhost」 になるため、ホストマシン（Macなど）からアクセスできない<br>
つまり<br>
設定：アクセスできる対象<br>
-b 127.0.0.1：コンテナ内のみ<br>
-b 0.0.0.0：外部（ホスト）からもアクセス可能！<br>
👉 -b 0.0.0.0 にすると「どこからでも接続を受け付ける」ようになるので、ホストから http://localhost:3000 でアクセスできるようになる

### 🌐 -p 3000 の意味（ポート指定）
デフォルトのRailsサーバーのポートは 3000 だが、Procfile.dev に明示的に -p 3000 を書くことで、Dockerの ports: 設定と一致する<br>
たとえば docker-compose.yml でこう書いてあるなら：
```yaml
ports:
  - "3000:3000"
```
Rails側も -p 3000 で合わせておく必要がある
