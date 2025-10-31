# マイページにリポストした投稿を表示

## ✅ 行ったこと

- マイページにリポストした投稿を表示させるようにした

## ①UsersController#showの編集
```
def show
  @user = User.find(params[:id])

  own_posts = @user.posts
                   .includes(:user)
                   .select('posts.*, posts.created_at AS acted_at, NULL::integer AS actor_id, \'post\' AS kind')

  reposted_posts = Post.joins(:reposts)
                       .where(reposts: { user_id: @user.id })
                       .includes(:user)
                       .select('posts.*, reposts.created_at AS acted_at, reposts.user_id AS actor_id, \'repost\' AS kind')

  @timeline = (own_posts.to_a + reposted_posts.to_a)
                .sort_by { |r| r.acted_at }
                .reverse
end

```

### 1. ユーザーの特定
```
@user = User.find(params[:id])
```
/users/:id の :id から、マイページの主役となるユーザーを取得する

### 2. ユーザー自身の投稿
```
  own_posts = @user.posts
                   .includes(:user)
                   .select('posts.*, posts.created_at AS acted_at, NULL::integer AS actor_id, \'post\' AS kind')
```
- @user.posts で このユーザーの投稿だけを取得
- .includes(:user) はビューで post.user.name を出す時のN+1クエリ防止
- .select(...) で「タイムラインに並べるための共通列」を付与：
  - acted_at：このレコードを並べるときに使う“出来事の時刻”、自分の投稿は 投稿時刻（posts.created_at）
  - actor_id：この出来事を起こした人。自分の投稿は“行為者”の概念が不要なので NULL
  - kind：種類フラグ。"post" を文字列で持たせる（ビューで分岐に使う）

### 3. ユーザーがリツイートした投稿
```
  reposted_posts = Post.joins(:reposts)
                       .where(reposts: { user_id: @user.id })
                       .includes(:user)
                       .select('posts.*, reposts.created_at AS acted_at, reposts.user_id AS actor_id, \'repost\' AS kind')
```
- Post.joins(:reposts) で リツイート経由の投稿を取得
- where(reposts: { user_id: @user.id }) で「このユーザーがRTした行だけ」に限定する
- .includes(:user) は元投稿作者のN+1防止。
- .select(...) の意味：
  - acted_at：RTした時刻（reposts.created_at）
    → 投稿の作成日ではなく“あなたがRTした瞬間”で並べられる
  - actor_id：RTした人のid（= @user.id）
    → ビューで「あなたがリツイート」「○○さんがリツイート」と表示できる
  - kind："repost" フラグ

### 4. 二つの集合を一つにまとめる
```
@timeline = (own_posts.to_a + reposted_posts.to_a)
              .sort_by { |r| r.acted_at }
              .reverse
```
- +：配列同士を結合。
- sort_by { |r| r.acted_at }：**出来事時刻（acted_at）**で昇順ソート。
- .reverse：新しい順（降順）に並べ替える
  → タイムラインとして自然な順序になる

## ②show.html.erbのビューを変更
```
<h1 class="h4 mb-3"><%= @user.name %></h1>

  <h2 class="h5 mb-3">つぶやき一覧</h2>
  <% if @timeline.present? %>
    <% @timeline.each do |item| %>
      <div id="<%= dom_id(item) %>" class="card mb-3 position-relative">
        <div class="card-body position-relative">
          <% if item.kind == "repost" %>
            <div class="small text-success mb-1">
              <i class="bi bi-arrow-repeat"></i>
              リツイートしました
            </div>
          <% end %>

          <div class="small text-muted mb-1">
            <strong><%= item.user&.name || "Unknown" %></strong>
            ・ <%= l(item.created_at, format: :short) rescue item.created_at.to_s %>
          </div>

          <p class="card-text mb-2"><%= simple_format(h(item.body)) %></p>

          <div class="mt-2 pt-2 border-top position-relative z-3 d-flex justify-content-between align-items-center">
            <div class="d-inline-flex align-items-center gap-3">
              <%= render "posts/like_button", post: item %>
              <%= render "posts/repost_button", post: item %>
            </div>
            <div class="d-inline-flex gap-2">
              <% if logged_in? && item.user_id == current_user.id %>
                <%= link_to "編集", edit_post_path(item), class: "btn btn-outline-success btn-sm" %>
                <%= button_to "削除", post_path(item), method: :delete,
                      data: { turbo_confirm: "このつぶやきを削除します。よろしいですか？" },
                      class: "btn btn-outline-danger btn-sm" %>
              <% end %>
            </div>
          </div>
          <%= link_to "", post_path(item), class: "stretched-link" %>
        </div>
      </div>
    <% end %>
  <% else %>
    <div class="text-muted">投稿はまだありません。</div>
  <% end %>
```

### 1. @posts → @timeline に変更
```
<% if @timeline.present? %>
  <% @timeline.each do |item| %>
```
- 単に自分の投稿だけでなく、リツイートもまとめて表示するために変更
- コントローラで @timeline に「自分の投稿」と「リツイート」を統合した配列を用意しているので、ビュー側もそれに合わせてループ変数を item に変更

### 2. リツイート専用の表示ブロックを追加
```
<% if item.kind == "repost" %>
  <div class="small text-success mb-1">
    <i class="bi bi-arrow-repeat"></i>
    リツイートしました
  </div>
<% end %>
```
- @timeline 内の各アイテムは「kind」という属性を持っており、"post" or "repost" で種類を区別できるようにしてある
- ここでは item.kind == "repost" のときだけ、“リツイートしました” の帯（＋矢印アイコン）を表示する

  → これで通常投稿とリツイートを視覚的に区別できる

### 3. 各カード内の本文を表示
```
<strong><%= item.user&.name || "Unknown" %></strong>
・ <%= l(item.created_at, format: :short) rescue item.created_at.to_s %>
```
以前は post.user や post.created_at を使っていたが、変数名を item に統一する

### 4. いいね・リツイートボタンを並列表示
```
<div class="d-inline-flex align-items-center gap-3">
  <%= render "posts/like_button", post: item %>
  <%= render "posts/repost_button", post: item %>
</div>
```
ここでも変数を post: → item: に変更

### 5. 詳細ページへのリンク
```
<%= link_to "", post_path(item), class: "stretched-link" %>
```
ここでもpost_path(post)をpost_path(item)に変更

