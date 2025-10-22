# 投稿の削除機能実装

## ✅ 行ったこと

- つぶやいた投稿を削除する機能を追加した

## ①ルーティングにdestroyを追加
```
# config/routes.rb
resources :posts, only: [:index, :new, :create, :show, :destroy]
```

## ②モデルの関連（すでに記述済み）
```
# app/models/post.rb
class Post < ApplicationRecord
  belongs_to :user
  has_many :likes, dependent: :destroy   # ← これで投稿削除時にlikesも削除
  ...
end
```

## ③posts_contorollerを編集
```
class PostsController < ApplicationController
  include ActionView::RecordIdentifier

  before_action :authenticate_user!, only: [:new, :create, :destroy]     # ← destroy も保護
  before_action :set_post, only: [:show, :destroy]                        # ← 取得共通化
  before_action :authorize_owner!, only: [:destroy]                       # ← 自分の投稿だけ削除可

  def index
    @posts = Post.includes(:user).order(created_at: :desc)
  end

  def new
    @post = Post.new
  end

  def create
    @post = current_user.posts.build(post_params)
    if @post.save
      redirect_to posts_path, notice: "投稿しました"
    else
      flash.now[:alert] = "投稿に失敗しました"
      render :new, status: :unprocessable_entity
    end
  end

  def show; end # ← before_actionでset_postしているので短縮化

  def destroy
    @post.destroy
    respond_to do |format|
      format.html { redirect_to posts_path, notice: "投稿を削除しました" }
      format.turbo_stream  # ← destroy.turbo_stream.erb に委譲（下で作る）
    end
  end

  private

  def set_post
    @post = Post.find(params[:id])
  end

  def authorize_owner!
    redirect_to posts_path, alert: "権限がありません" unless @post.user_id == current_user&.id
  end

  def post_params
    params.require(:post).permit(:body)
  end
end
```

## ④showで詳細画面に削除ボタン装
```
<div class="container mt-5">
  <div class="card">
    <div class="card-body">
      <div class="small text-muted mb-1">
        <strong><%= @post.user&.name || "Unknown" %></strong>
        ・ <%= l(@post.created_at, format: :short) rescue @post.created_at.to_s %>
      </div>

      <p class="card-text mb-2"><%= simple_format(h(@post.body)) %></p>

      <!-- 本文直下のフッター行：いいね＋（自分のときだけ）削除 -->
      <div class="mt-2 pt-2 border-top d-flex align-items-center gap-3">
        <%= render "posts/like_button", post: @post %>

        <% if logged_in? && @post.user_id == current_user.id %>
          <%= button_to "削除",
                post_path(@post),
                method: :delete,
                data: { turbo_confirm: "この投稿を削除します。よろしいですか？" },
                class: "btn btn-outline-danger btn-sm" %>
        <% end %>
      </div>
    </div>
  </div>

  <div class="mt-3">
    <%= link_to "一覧に戻る", posts_path, class: "btn btn-outline-secondary" %>
  </div>
</div>

```

## ⑤indexで一覧画面に削除ボタンを追加
```
<div class="container mt-5">
  <div class="d-flex justify-content-between align-items-center mb-3">
    <h1 class="mb-0">つぶやき一覧</h1>
    <% if logged_in? %>
      <%= link_to "新規投稿", new_post_path, class: "btn btn-outline-primary" %>
    <% end %>
  </div>

  <% @posts.each do |post| %>
    <div id="<%= dom_id(post) %>" class="card mb-3 position-relative">
      <div class="card-body position-relative">
        <div class="small text-muted mb-1">
          <strong><%= post.user&.name || "Unknown" %></strong>
          ・ <%= l(post.created_at, format: :short) rescue post.created_at.to_s %>
        </div>
        <p class="card-text mb-2"><%= simple_format(h(post.body)) %></p>

        <!-- 本文直下のフッター行：いいね＋（自分のときだけ）削除 -->
        <div class="mt-2 pt-2 border-top position-relative z-3">
          <%= render "posts/like_button", post: post %>                      # ← ❌post: @post
          <% if logged_in? && post.user_id == current_user.id %>             # ← ❌<% if logged_in? && @post.user_id == current_user.id %>
            <%= button_to "削除",
                  post_path(post),
                  method: :delete,
                  data: { turbo_confirm: "この投稿を削除します。よろしいですか？" },
                  class: "btn btn-outline-danger btn-sm mt-2" %>
          <% end %>
        </div>
        <%= link_to "", post_path(post), class: "stretched-link" %>
      </div>
    </div>
  <% end %>
</div>
```

### エラー発生❗️
上記の❌で示しているように、@postをindexで用いるとエラーが出る

#### ローカル変数（post）
```
<% @posts.each do |post| %>
  ...
  post.user_id
<% end %>
```

- post は each のブロック変数
- このブロックの中でだけ有効
- つまり「いまループしてる1件の投稿」を指す

#### インスタンス変数（@post）

- コントローラでセットした値を ビュー全体で使える
- 例: PostsController#show で
  ```
  @post = Post.find(params[:id])
  ```
  としていれば、show.html.erb 内で @post が使える
- でも index アクションでは @posts は用意してても、@post は用意してない
  → だから @post は nil で、「nil.user_id」となってエラーになった

#### 今回のケース

一覧(index)では「繰り返し変数 post」を使わないといけない<br>
詳細(show)では「コントローラでセットした @post」を使う<br>

#### エラーの解決まとめ

- @xxx = インスタンス変数（コントローラからビュー全体に渡される）
- xxx = ローカル変数（eachブロックなど、そのスコープ内だけで有効）

indexはループだから post、showは単体だから @post を使う、という違い

## ⑥delete後にリロードなしでその場で投稿が消えるようにする

### カード（消す対象）に一意なIDを付ける
index.html.erb の各カードの外側に id="<%= dom_id(post) %>" を付ける
```
<% @posts.each do |post| %>
  <div id="<%= dom_id(post) %>" class="card mb-3 position-relative">                      # ← ココ
    ...
    <div class="mt-2 pt-2 border-top position-relative z-3 d-flex align-items-center gap-3">
      <%= render "posts/like_button", post: post %>

      <% if logged_in? && post.user_id == current_user.id %>
        <%= button_to "削除",
              post_path(post),
              method: :delete,
              data: { turbo_confirm: "この投稿を削除します。よろしいですか？" },
              class: "btn btn-outline-danger btn-sm" %>
      <% end %>

      <%= link_to "", post_path(post), class: "stretched-link" %>
    </div>
  </div>
<% end %>
```

### コントローラで Turbo 応答を用意
PostsController#destroy に format.turbo_stream を追加
```
def destroy
  @post.destroy
  respond_to do |format|
    format.html { redirect_to posts_path, notice: "投稿を削除しました" } # 非Turboのフォールバック
    format.turbo_stream                                                  # ← 次のテンプレに委譲
  end
end
```

### Turbo Stream テンプレートでDOMから削除
app/views/posts/destroy.turbo_stream.erb を作成
```
<%= turbo_stream.remove dom_id(@post) %>
```