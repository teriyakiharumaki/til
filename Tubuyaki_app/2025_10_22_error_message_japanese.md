# エラーメッセージの日本語化

## ✅ 行ったこと

- エラーメッセージの日本語化を行った

## コードの記入
config/application.rbに以下の記述[1]
```
class Application < Rails::Application
    # 省略
    config.i18n.default_locale = :ja # <-追加
    
    # locales直下にファイルを置かずフォルダ分けする場合はパスを追加
    # config.i18n.load_path += Dir[Rails.root.join("config", "locales", "**", "*.{rb,yml}").to_s] 
 end
```

## エラー
[![Image from Gyazo](https://i.gyazo.com/98e6ef2f8551de85580f0a4a527f89e6.png)](https://gyazo.com/98e6ef2f8551de85580f0a4a527f89e6)

ターミナル上
```
tubuyaki-web-1  | 01:36:31 web.1  | ActionView::Template::Error (:ja is not a valid locale)
tubuyaki-web-1  | 01:36:31 web.1  | Caused by: I18n::InvalidLocale (:ja is not a valid locale)
tubuyaki-web-1  | 01:36:31 web.1  | 
tubuyaki-web-1  | 01:36:31 web.1  | Information for: ActionView::Template::Error (:ja is not a valid locale):
tubuyaki-web-1  | 01:36:31 web.1  |     2:   <% if @post.errors.any? %>
tubuyaki-web-1  | 01:36:31 web.1  |     3:     <div class="alert alert-danger">
tubuyaki-web-1  | 01:36:31 web.1  |     4:       <ul class="mb-0">
tubuyaki-web-1  | 01:36:31 web.1  |     5:         <% @post.errors.full_messages.each do |m| %>
tubuyaki-web-1  | 01:36:31 web.1  |     6:           <li><%= m %></li>
tubuyaki-web-1  | 01:36:31 web.1  |     7:         <% end %>
tubuyaki-web-1  | 01:36:31 web.1  |     8:       </ul>
tubuyaki-web-1  | 01:36:31 web.1  |   
tubuyaki-web-1  | 01:36:31 web.1  | app/views/posts/_form.html.erb:5
tubuyaki-web-1  | 01:36:31 web.1  | app/views/posts/_form.html.erb:1
tubuyaki-web-1  | 01:36:31 web.1  | app/views/posts/new.html.erb:3
tubuyaki-web-1  | 01:36:31 web.1  | app/controllers/posts_controller.rb:22:in `create'
tubuyaki-web-1  | 01:36:31 web.1  | 
tubuyaki-web-1  | 01:36:31 web.1  | Information for cause: I18n::InvalidLocale (:ja is not a valid locale):
tubuyaki-web-1  | 01:36:31 web.1  |   
tubuyaki-web-1  | 01:36:31 web.1  | app/views/posts/_form.html.erb:5
tubuyaki-web-1  | 01:36:31 web.1  | app/views/posts/_form.html.erb:1
tubuyaki-web-1  | 01:36:31 web.1  | app/views/posts/new.html.erb:3
tubuyaki-web-1  | 01:36:31 web.1  | app/controllers/posts_controller.rb:22:in `create'
```

## 💥エラーの意味
```
I18n::InvalidLocale: :ja is not a valid locale
```
Rails が「:ja 用の翻訳データ（ja.yml）」を探したけど、見つけられなかった

## ✅ 解決方法
**Rails公式の日本語化ファイルを導入**

Railsのエラーメッセージなどを日本語化する定番 gem rails-i18n を追加

### 1️⃣ Gemfile に追記
gem 'rails-i18n'

### 2️⃣ 反映
bundle install

### 3️⃣ 設定を確認

すでに config/application.rb に書いてあるこれ
```
config.i18n.default_locale = :ja
```

### 導入した結果
[![Image from Gyazo](https://i.gyazo.com/280eb3d7f871064981ee0c68aa62e97a.png)](https://gyazo.com/280eb3d7f871064981ee0c68aa62e97a)

## 日本語化をさらに細かく
先頭の属性名は自前で用意する<br>
config/locales/ja.yml
```
ja:
  activerecord:
    models:
      post: 投稿
    attributes:
      post:
        body: 本文
```

### 導入した結果
[![Image from Gyazo](https://i.gyazo.com/7703145b73135f6da1fac384badb09e1.png)](https://gyazo.com/7703145b73135f6da1fac384badb09e1)

## 参考サイト
[1](https://zenn.dev/kazmaendo/articles/63a9104a7a41f5)