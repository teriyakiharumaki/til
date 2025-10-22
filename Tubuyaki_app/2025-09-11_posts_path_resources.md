# posts_pathについて

## ✅ 行ったこと

- ログイン後の遷移をつぶやき一覧にする際に、posts_pathでなぜ実装できるのかについて考えた

## 疑問
routeでは
```
resources :posts, only: [:index, :new, :create]
```
で実装しており、posts/indexで一覧ページへの実装を行うかと思っていたら、
```
  def create
    user = User.find_by(email: params[:email])
    if user&.authenticate(params[:password])
      session[:user_id] = user.id
      redirect_to posts_path, notice: "ログインに成功しました！"
```
このようにposts_pathで実装できた

## 解説

### resources :posts がやってること
```
resources :posts, only: [:index, :new, :create]
```
これを書くと、Railsが「RESTfulなルート」を自動生成される

| HTTPメソッド | パス | コントローラ#アクション | 名前付きルートヘルパー |
| ---- | ---- | ---- | ---- |
| GET | /posts | posts#index | posts_path |
| GET | /posts/new | posts#new | new_post_path |
| POST | /posts | posts#create | posts_path |

### posts_path とは？

- 名前付きルート = ルーティングに自動で用意される「パスを返すメソッド」。
- posts_path → /posts というURLを返す。
- このパスに GET でアクセスした場合、ルーティング定義によって posts#index が呼ばれる。
- 同じ /posts に POST でアクセスした場合は posts#create が呼ばれる。

👉 つまり 「パス（/posts）」は同じでも、HTTPメソッドによって呼ばれるアクションが違う という仕組み。

### なぜ posts#index を直接書かないのか？
RailsではビューやコントローラからURLを呼びたいときに
```
posts_path   # => "/posts"
new_post_path # => "/posts/new"
```
みたいな「名前付きルートヘルパー」を使うのが慣習。<br>
もし redirect_to posts_path と書けば → GET /posts が呼ばれるので → posts#index に到達する。

### 結論
💡つまり「posts_path = posts#index」ではなく、<br>
「posts_path = /posts というURL」であり、<br>
「そのURLに GET で行けば結果的に posts#index が呼ばれる」 という関係
