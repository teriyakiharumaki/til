# いいねボタン解説

## ✅ 行ったこと

- いいねボタンのルーティングについて解説

## ルーティング
```
resources :posts, only: [:index, :new, :create, :show] do
  post :like, on: :member   # POST /posts/:id/like → posts#like
end
```

### 1. resources :posts

これは「postsコントローラに対して、RESTfulなルーティングをまとめて作る」Railsのマクロ<br>
ここでは only: を指定してるので、生成されるのはこの4つ：

- GET /posts → posts#index
- GET /posts/new → posts#new
- POST /posts → posts#create
- GET /posts/:id → posts#show

### 2. do ... end ブロック

resources にブロックを渡すと、その中で「追加のルート」を定義できる。<br>
つまり「投稿（post）に紐づいたカスタムアクション」を定義したいときに使う。

### 3. post :like, on: :member

- post … HTTPメソッドは POST を使うよ、という指定
- :like … アクション名（→ コントローラの like メソッドが呼ばれる）
- on: :member … そのリソース（= 各post）に対して実行するアクションという意味
👉 これによって Rails が自動的にルートを作る：
```
POST /posts/:id/like → posts#like
```
:id が付くのは member 指定だから<br>
（もし on: :collection とすると /posts/like になり、idなしで「全体に対する操作」になる）

### 4. 名前付きルート
これを追加すると、Railsはヘルパーメソッドも作ってくれる
```
like_post_path(@post)
```
という形で /posts/:id/like のURLを呼べる。<br>
だからビューで
```
<%= button_to "いいね", like_post_path(@post), method: :post %>
```
と書けば、自動的に POST /posts/:id/like が飛び、PostsController#like が呼ばれる
