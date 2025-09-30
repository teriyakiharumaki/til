# いいねボタンを一覧画面でも実装

## ✅ 行ったこと

- いいねボタンを一覧画面でも実装した
- いいねボタンの位置をつぶやきの本文と区切るため、フッターとして実装した

## indexの修正
```
<% @posts.each do |post| %>
  <div class="card mb-3 position-relative">
    <div class="card-body position-relative">
      <div class="small text-muted mb-1">
        <strong><%= post.user&.name || "Unknown" %></strong>
        ・ <%= l(post.created_at, format: :short) rescue post.created_at.to_s %>
      </div>
      <p class="card-text mb-2"><%= simple_format(h(post.body)) %></p>
      <div class="mt-2 pt-2 border-top position-relative z-3">
        <%= render "posts/like_button", post: post %>
      </div>
      <%= link_to "", post_path(post), class: "stretched-link" %>
    </div>
  </div>
<% end %>

```

## indexの各部分解説

### 全体
```
<% @posts.each do |post| %>
  <div class="card mb-3 position-relative">
    <div class="card-body position-relative">
      ...
    </div>
  </div>
<% end %>
```
- @posts.each → コントローラーで用意した投稿一覧を1件ずつ処理
- card / card-body → BootstrapのカードUI。投稿ごとにカードを1枚描画
- position-relative → 後で stretched-link を重ねるための必須設定

### 投稿者名・日付
```
<div class="small text-muted mb-1">
  <strong><%= post.user&.name || "Unknown" %></strong>
  ・ <%= l(post.created_at, format: :short) rescue post.created_at.to_s %>
</div>
```
- post.user&.name || "Unknown" → 投稿にユーザーが紐づいていれば名前、なければ "Unknown" を表示
- l(post.created_at, format: :short) → Railsの l (localize) で日付をフォーマット
  - rescue post.created_at.to_s → ロケールが未設定のときでも安全に文字列化
- small text-muted → 小さめフォント＋薄いグレー
- mb-1 → 下に少し余白


### 本文
```
<p class="card-text mb-2"><%= simple_format(h(post.body)) %></p>
```
- h(post.body) → 投稿本文をHTMLエスケープして安全に。
- simple_format → 改行を `<br>` に変換しつつ `<p>`で囲むRailsヘルパー
- card-text → Bootstrapカード用テキストスタイル
- mb-2 → 本文の下に余白

### フッター行（いいねボタン）
```
<div class="mt-2 pt-2 border-top position-relative z-3">
  <%= render "posts/like_button", post: post %>
</div>
```
- mt-2 → 本文から少し離す
- pt-2 border-top → 上に線を入れて「本文とアクションの区切り」に
- position-relative z-3 → stretched-link（後述）のオーバーレイより前面に出す。これがないとボタンが押せずに詳細へ飛んでしまう
- render "posts/like_button", post: post → ハートアイコン＋カウントのパーシャルを呼び出し

### カード全体のリンク化
```
<%= link_to "", post_path(post), class: "stretched-link" %>
```
- 中身のない link_to だけど、stretched-link クラスのおかげで カード全体がリンク化
- クリックすると posts#show に遷移
- 透明なリンクを背面に敷く仕組みなので、z-3 を付けた「いいねボタン」だけは上に出て押せる

## showの修正

```
<div class="container mt-5">
  <div class="card">
    <div class="card-body">
      <div class="small text-muted mb-1">
        <strong><%= @post.user&.name || "Unknown" %></strong>
        ・ <%= l(@post.created_at, format: :short) rescue @post.created_at.to_s %>
      </div>

      <p class="card-text mb-2"><%= simple_format(h(@post.body)) %></p>
      <div class="mt-2 pt-2 border-top position-relative z-3">
        <%= render "posts/like_button", post: @post %>
      </div>
    </div>
  </div>
  <div class="mt-3">
    <%= link_to "一覧に戻る", posts_path, class: "btn btn-outline-secondary" %>
  </div>
</div>
```
いいねボタンのフッター化を行なったのみ