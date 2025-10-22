# いいねボタン実装

## ✅ 行ったこと

- つぶやき詳細画面のみで押せるいいねボタンを実装した

## 実装

### 1. ルーティングに like を追加
```
# config/routes.rb
resources :posts, only: [:index, :new, :create, :show] do
  post :like, on: :member   # POST /posts/:id/like → posts#like
end
```

### 2. コントローラにアクションを追加 
```
# app/controllers/posts_controller.rb
def like
  @post = Post.find(params[:id])
  @post.increment!(:likes_count)
  respond_to do |format|
    format.html { redirect_back fallback_location: post_path(@post), notice: "いいねしました" }
    format.turbo_stream do
      flash.now[:notice] = "いいねしました"
      render turbo_stream: [
        turbo_stream.replace(
          dom_id(@post, :likes),
          partial: "posts/likes",
          locals: { post: @post }
        ),
        turbo_stream.replace(
          "flash",
          partial: "shared/flash"
        )
      ]
    end
  end
end
```

### 3. パーシャルファイルでいいねボタンを実装
```
<!-- app/views/posts/_likes.html.erb -->
<span id="<%= dom_id(post, :likes) %>" class="badge bg-light text-dark">
  いいね <%= post.likes_count %>
</span>
```

### 4. ビューにボタンを設置
一覧は「カード全体がリンク」なので、ボタンを中に入れるとリンクの中にフォームが入ってしまいNG<br>
まずは詳細ページ（show）にだけ いいねボタンを置く
```
<!-- app/views/posts/show.html.erb（抜粋） -->
<p class="card-text mb-2"><%= simple_format(h(@post.body)) %></p>

<%= render "posts/likes", post: @post %>

<% if logged_in? %>
  <div class="mt-3">
    <%= button_to "いいね", like_post_path(@post), method: :post, class: "btn btn-outline-primary btn-sm" %>
  </div>
<% end %>
```

### エラー発生❗️
```
NoMethodError in PostsController#like
undefined method dom_id' for an instance of 
PostsController 
Extracted source (around line #35):
33 render turbo_stream: [
34  turbo_stream.replace(
35    dom_id(@post, :likes),
36    partial: "posts/likes",
37    locals: { post: @post }
38   ),
```
原因→dom_id はビュー用ヘルパーで、コントローラ内ではそのまま使えないから<br>
解決：
```
# app/controllers/posts_controller.rb
class PostsController < ApplicationController
  include ActionView::RecordIdentifier   # ← これを追加！

  def like
    @post = Post.find(params[:id])
    @post.increment!(:likes_count)
    respond_to do |format|
      format.html { redirect_back fallback_location: post_path(@post), notice: "いいねしました" }
      format.turbo_stream do
        flash.now[:notice] = "いいねしました"
        render turbo_stream: [
          turbo_stream.replace(
            dom_id(@post, :likes),                    # ← そのまま使えるようになる
            partial: "posts/likes",
            locals: { post: @post }
          ),
          turbo_stream.replace("flash", partial: "shared/flash")
        ]
      end
    end
  end
end
```