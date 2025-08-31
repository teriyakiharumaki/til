# サインアップ機能

## ✅ 行ったこと

- サインアップ機能の実装

## bcrypt（gem）の導入

```
gem "bcrypt", "~> 3.1"
```
```
bundle install
```
これで導入

## Userモデルの作成

```
rails g model User name:string email:string password_digest:string
```
これで以下のマイグレーションファイルが作成
```
class CreateUsers < ActiveRecord::Migration[8.0]
  def change
    create_table :users do |t|
      t.string :name
      t.string :email
      t.string :password_digest

      t.timestamps
    end
  end
end
```
マイグレーションを実行することで、usersテーブルを作成
```
rails db:migrate
```
できたカラムは

- ユーザの名前：email:string
- ユーザのメールアドレス：email:string
- ログイン認証のためのユーザパスワード（暗号化される）：password_digest:string

になっている

### password_digestとは[1]
暗号の分野では、暗号化（ダイジェスト）された文字列をメッセージダイジェストと呼びます。<br>
メッセージダイジェストは、元の文字列に戻すことができません。<br>
https://wa3.i-3-i.info/word15954.html<br>
has_secure_passwordではデータベース内のpassword_digestという属性に保存できるようになります。(passwordではない点に注意してください)<br>

### モデル[1]
user.rbモデルファイルにhas_secure_password を記述
```
class User < ApplicationRecord
  has_secure_password  # ← これで password / password_confirmation が使える
  validates :name,  presence: true
  validates :email, presence: true, uniqueness: true
end
```
こうするだけで、下記の暗号化に関する便利な機能が提供されます。<br>

- users テーブルにパスワードを保存するとき、パスワードを暗号化して保存してくれる
- ログイン用のフォーム（View）にpasswordとpassword_confirmationという変数をモデルに追加すれば暗号化してDBに保存してくれる
- ログイン認証用のメソッドであるauthenticateが使用できる

簡単に言えば、password_digestカラムを用意して、モデルファイルにhas_secure_passwordを記述すれば、ログインの機能をよしなにやってくれるやつです。

## サインアップ機能の追加（UsersController）
UsersControllerの作成
```
rails g controller Users new create
```
routes.rbに追加
```
resources :users, only: [:new, :create]
get  "signup", to: "users#new"
```
app/controllers/users_controller.rbを編集
```
class UsersController < ApplicationController
  def new
    @user = User.new
  end
  def create
    @user = User.new(user_params)
    if @user.save
      session[:user_id] = @user.id
      redirect_to root_path, notice: "Signed up!"
    else
      render :new, status: :unprocessable_entity
    end
  end
  private
  def user_params
    params.require(:user).permit(:name, :email, :password, :password_confirmation)
  end
end
```
app/views/users/new.html.erbに登録用フォームの追加
```
<h1 class="text-xl font-bold mb-4">Sign up</h1>
<%= form_with model: @user, url: users_path, class: "space-y-3" do |f| %>
  <div><%= f.label :name %><%= f.text_field :name, class: "border p-2 w-full" %></div>
  <div><%= f.label :email %><%= f.email_field :email, class: "border p-2 w-full" %></div>
  <div><%= f.label :password %><%= f.password_field :password, class: "border p-2 w-full" %></div>
  <div><%= f.label :password_confirmation %><%= f.password_field :password_confirmation, class: "border p-2 w-full" %></div>
  <%= f.submit "Create account", class: "border px-4 py-2" %>
<% end %>
```

## サインアップの流れについての解説

### 🔄 サインアップ処理の流れ

1. ユーザーが /signup にアクセス

- routes.rb で get "signup", to: "users#new"
- UsersController#new が呼ばれる
- 中で @user = User.new → 空のUserオブジェクトを用意
- new.html.erb に渡されて フォームが表示される

2. ユーザーがフォームを送信

- form_with model: @user, url: users_path なので、送信先は POST /users
- ルーティングが resources :users, only: [:create] を持ってるので、UsersController#create に到達

3. UsersController#create の中身

- @user = User.new(user_params) → フォームの値を入れてUserを生成
- if @user.save でバリデーションチェック & DB保存を試みる
  - 成功 → session[:user_id] = @user.id でログイン状態にする
    その後 redirect_to root_path, notice: "Signed up!" でトップへリダイレクト
  - 失敗 → render :new でフォームを再表示（エラーメッセージも出せる）

### 💡 new と create の役割の違い

- new
  - サインアップ画面（フォーム）を「表示するだけ」
  - 中で @user = User.new して、フォームビルダー（form_with）に渡すための入れ物を用意する

- create
  - 「フォームから送られてきた値」を受け取って実際にユーザーを「作成（保存）」する処理
  - DBに書き込みが発生する

👉 つまり
- new = 入力画面を表示する
- create = 入力を受け取って保存する

この2つがセットで「サインアップ機能」を成立させてる。

### 実際のイメージ

- /signup へアクセスすると new が呼ばれて「サインアップ画面」が出る
- そこで入力して「送信」を押すと create が呼ばれて「保存処理」が走る
- 成功すればトップページへ、失敗すれば再度「new」を表示

## なんでフォームを送信すると create が呼ばれるのか？

### フォームの送信先はどう決まる？
```
<%= form_with model: @user, url: users_path %>
```
- model: @user があるから、Railsは「Userモデルのルーティング」を見に行く。
- url: users_path は、ルーティングヘルパー users_path を指定している。

👉 ここで生成されるフォームタグは実際にはこうなる：
```
<form action="/users" method="post">
```

### /users (POST) にアクセスしたときのルーティング
```
resources :users, only: [:new, :create]
```
これがあると、Railsは以下のルートを自動生成する：
- GET /users/new → users#new
- POST /users → users#create

👉 つまり 「/users に POST が来たら create を呼ぶ」 というルールが routes.rb で出来上がっている

### ユーザーが「送信」ボタンを押すと
- ブラウザがフォームの `<form>`情報を読み取って、action="/users" method="post" に従い、サーバーにリクエストを投げる。
- サーバー(Rails)は routes を見て、「/users に POST → UsersController#create だ！」と判別する。
- その結果、create メソッドが呼ばれる。

### 結論
👉 つまり、「ボタンを押したから create が呼ばれる」のではなく、
「ボタンを押した → POST /users リクエストが送られる → routes が create に振り分ける」 という流れ

## resources の役割

### モヤモヤしてるポイント
「resources :users, only: [:new, :create] のルーティングにPOST /usersが来た場合は自動的にUsersController#createへ飛ばすようにできてるってこと？ それはresourcesの機能？」

### resourcesについて
resources :users, only: [:new, :create] と書くと、Railsが「RESTfulルーティング」という決まりに従ってルートを自動生成してくれる。<br>
この場合、2つだけルートが作られる：<br>

| HTTPメソッド | パス | コントローラ#アクション | 用途 |
| ---- | ---- | ---- | ---- |
| GET | /users/new | users#new | 新規ユーザー入力画面 |
| POST | /users | users#create | 新規ユーザー登録処理 |

### 🔹 仕組み

- resources は「URLとHTTPメソッドの組み合わせ」を「コントローラ#アクション」にマッピングする。
- だから POST /users が来たら、自動的に UsersController#create に振り分ける ようになっている。
- これが Rails の ルーティング規約（Convention over Configuration の一部）。

### まとめ
resources は「RESTfulなURL設計を自動で用意してくれるマクロ」だから、POST /users が users#create に飛ぶのはその仕組み。


## 参考サイト
[1](https://qiita.com/ryosuketter/items/805452b7e6bf9637cb57)<br>
resourcesについて→(https://pikawaka.com/rails/resources)