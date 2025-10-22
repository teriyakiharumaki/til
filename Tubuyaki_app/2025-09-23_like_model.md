# いいねボタンの作り直し

## ✅ 行ったこと

- いいねボタンのモデルを作り、機能を作り直す

## ① モデル & マイグレーション
コンテナ内でモデルを作成
```
rails g model Like user:references post:references
```
マイグレーションファイルを編集
```
# db/migrate/20250916182304_create_likes.rb
class CreateLikes < ActiveRecord::Migration[7.1]
  def change
    create_table :likes do |t|
      t.references :user, null: false, foreign_key: true
      t.references :post, null: false, foreign_key: true
      t.timestamps
    end
    add_index :likes, [:user_id, :post_id], unique: true
  end
end
```
コンテナ内でマイふレーション
```
rails db:migrate
```

## ② アソシエーション & counter_cache
```
# app/models/like.rb
class Like < ApplicationRecord
  belongs_to :user
  belongs_to :post, counter_cache: true
  validates :user_id, uniqueness: { scope: :post_id }
end
```
✅ belongs_to :user

- Like は 必ずどのユーザーのいいねか を持つ。
- likes.user_id カラムと users.id をつないで関連付け。

✅ belongs_to :post, counter_cache: true

- Like は 必ずどの投稿に対するいいねか を持つ。
- counter_cache: true を付けることで、Like を追加/削除するたびに posts.likes_count カラムを Railsが自動で増減して更新してくれる。<br>
→ これのおかげで「毎回 .likes.count を数えるSQL」を打たずに、カラムの数字だけで素早く表示できる。

✅ validates :user_id, uniqueness: { scope: :post_id }

- 「同じユーザーが同じ投稿に2回以上いいねできない」制約。
- 例: user_id=1, post_id=5 のLikeがすでにあるなら、もう一度作ろうとしたらエラーになる。
- DBのインデックス制約と合わせて、アプリとDB両方で多重押しを防止できる。

```
# app/models/post.rb
class Post < ApplicationRecord
  belongs_to :user
  has_many :likes, dependent: :destroy
  has_many :liked_users, through: :likes, source: :user

  validates :body, presence: true, length: { maximum: 280 }

  def liked_by?(user)
    return false unless user
    likes.exists?(user_id: user.id)
  end
end
```
✅ has_many :likes, dependent: :destroy

- 投稿は複数のLikeを持つ。
- dependent: :destroy → 投稿が削除されたら、関連するいいねもまとめて消す。

✅ has_many :liked_users, through: :likes, source: :user

- Post から「この投稿にいいねしているユーザー一覧」を直接取得できる関連。
- post.liked_users と書けば、ユーザーの配列が返る。
- through: :likes で中間テーブル（likes）を経由、source: :user で like → user に変換。

✅ def liked_by?(user)

- 引数で渡したユーザーがこの投稿を「いいね済み」か判定する便利メソッド。
- コントローラやビューで if post.liked_by?(current_user) みたいに書ける。

```
# app/models/user.rb
class User < ApplicationRecord
  has_secure_password
  has_many :posts, dependent: :destroy
  has_many :likes, dependent: :destroy
end
```
✅ has_many :likes, dependent: :destroy

- ユーザーは複数のいいねを持つ。
- ユーザー削除時にその人のいいねもまとめて消す。

＊アソシエーション：「Railsでモデル同士の関係（リレーション）をつなぐ仕組み全般」(belongs_to とか has_many とかを使ってモデル間の関係を定義したこと)

## ③ルーティング
```
# config/routes.rb
resources :posts, only: [:index, :new, :create, :show] do
  resource :like, only: [:create, :destroy]   # POST /posts/:post_id/like, DELETE /posts/:post_id/like
end
```

## ④コントローラ（LikesController）
コンテナ内でlikesコントローラーを作成
```
rails g controller Likes
```
内容
```
# app/controllers/likes_controller.rb
class LikesController < ApplicationController
  include ActionView::RecordIdentifier
  before_action :authenticate_user!
  before_action :set_post

  def create
    current_user.likes.find_or_create_by!(post: @post)
    respond_to do |format|
      format.html { redirect_back fallback_location: post_path(@post), notice: "いいねしました" }
      format.turbo_stream do
        flash.now[:notice] = "いいねしました"
        render turbo_stream: [
          turbo_stream.replace(dom_id(@post, :likes),      partial: "posts/likes",        locals: { post: @post }),
          turbo_stream.replace(dom_id(@post, :like_button), partial: "posts/like_button", locals: { post: @post }),
          turbo_stream.replace("flash", partial: "shared/flash")
        ]
      end
    end
  end

  def destroy
    current_user.likes.where(post: @post).destroy_all
    respond_to do |format|
      format.html { redirect_back fallback_location: post_path(@post), notice: "いいねを取り消しました" }
      format.turbo_stream do
        flash.now[:notice] = "いいねを取り消しました"
        render turbo_stream: [
          turbo_stream.replace(dom_id(@post, :likes),       partial: "posts/likes",        locals: { post: @post }),
          turbo_stream.replace(dom_id(@post, :like_button),  partial: "posts/like_button", locals: { post: @post }),
          turbo_stream.replace("flash", partial: "shared/flash")
        ]
      end
    end
  end

  private
  def set_post
    @post = Post.find(params[:post_id])
  end
end
```

### ⑤ ビュー（パーシャル2つ）
- いいね数（既に作成済みならOK）
```
<!-- app/views/posts/_likes.html.erb -->
<span id="<%= dom_id(post, :likes) %>" class="badge bg-light text-dark">
  いいね <%= post.likes_count %>
</span>
```

- いいねボタン（状態で出し分け）
```
<!-- app/views/posts/_like_button.html.erb -->
<div id="<%= dom_id(post, :like_button) %>">
  <% if logged_in? %>
    <% if post.liked_by?(current_user) %>
      <%= button_to "いいねを取り消す", post_like_path(post), method: :delete,
            class: "btn btn-outline-secondary btn-sm", data: { turbo: true } %>
    <% else %>
      <%= button_to "いいね", post_like_path(post), method: :post,
            class: "btn btn-outline-primary btn-sm", data: { turbo: true } %>
    <% end %>
  <% else %>
    <%= link_to "ログインしていいね", login_path, class: "btn btn-outline-primary btn-sm" %>
  <% end %>
</div>
```

### ⑥ show.html の差し替え
```
<!-- app/views/posts/show.html.erb（抜粋） -->
<p class="card-text mb-2"><%= simple_format(h(@post.body)) %></p>

<%= render "posts/likes", post: @post %>
<%= render "posts/like_button", post: @post %>
```

### ⑦ 旧コードの整理
- posts_controller.rb の like アクションと、routes の post :like, on: :member は 削除
- 今後は LikesController#create/destroy のみを使います
- 既存の likes_count 直接 increment! は 不要（counter_cacheに任せる）
- PostsControllerのbefore_actionからlikeを消す（エラーが起こる）
