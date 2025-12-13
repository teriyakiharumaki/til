# tailwind　cssの導入

## ✅ 行ったこと

- tailwind cssの導入を行なった

## 　導入の流れ

0. `Gemfile`に以下を記述、`bundle install`（ここはすでに導入済）

```
gem "cssbundling-rails"
```

1. 以下を実行

```
bin/rails css:install:tailwind
```

2. 以下を package.json の scripts に追加する

```
"build:css": "tailwindcss -i ./app/assets/stylesheets/application.tailwind.css -o ./app/assets/builds/application.css --minify"
```
- グローバル or ローカルインストール済 CLI を直接実行

しかし、すでに以下を記述済
```
"build:css": "npx @tailwindcss/cli -i ./app/assets/stylesheets/application.tailwind.css -o ./app/assets/builds/application.css --minify"
```
- Tailwind CSS v4 の公式推奨 CLI を使った構文

この二つは、どちらも同じ意味で、CSS を Tailwind 経由でビルドするコマンド(実行エンジンが違う)

## 参考サイト
https://www.mof-mof.co.jp/tech-blog/2024/04/12/114220