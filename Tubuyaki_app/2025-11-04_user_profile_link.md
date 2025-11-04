# 投稿のユーザー名のリンク化

## ✅ 行ったこと

- 投稿のユーザー名のリンク化を行なった

## 投稿のユーザー名をリンクにする

### 一覧（app/views/posts/index.html.erb）
```
<div class="small text-muted mb-1 position-relative z-3">
  <% if post.user.present? %>
    <strong><%= link_to post.user.name, user_path(post.user), class: "text-decoration-none" %></strong>
  <% else %>
    <strong>Unknown</strong>
  <% end %>
  ・ <%= l(post.created_at, format: :short) rescue post.created_at.to_s %>
</div>
```

### 詳細（app/views/posts/show.html.erb）
```
<div class="small text-muted mb-1 position-relative z-3">
  <% if @post.user.present? %>
    <strong><%= link_to @post.user.name, user_path(@post.user), class: "text-decoration-none" %></strong>
  <% else %>
    <strong>Unknown</strong>
  <% end %>
  ・ <%= l(@post.created_at, format: :short) rescue @post.created_at.to_s %>
</div>
```

### ユーザーのページ（users/show.html.erb）
```
<div class="small text-muted mb-1 position-relative z-3">
  <% if item.user.present? %>
    <strong><%= link_to item.user.name, user_path(item.user), class: "text-decoration-none" %></strong>
  <% else %>
    <strong>Unknown</strong>
  <% end %>
  ・ <%= l(item.created_at, format: :short) rescue item.created_at.to_s %>
</div>
```

## 解説

一覧 / 詳細 / マイページ 共通の構造
```
<div class="small text-muted mb-1 position-relative z-3">
  <% if XXX.user.present? %>
    <strong>
      <%= link_to XXX.user.name, user_path(XXX.user), class: "text-decoration-none" %>
    </strong>
  <% else %>
    <strong>Unknown</strong>
  <% end %>
  ・ <%= l(XXX.created_at, format: :short) rescue XXX.created_at.to_s %>
</div>
```
### 各要素の意味

- small text-muted mb-1

  Bootstrapのユーティリティで控えめなメタ情報行（ユーザー名＋日時）の見た目にする

- position-relative z-3

  stretched-link 対策。カード全体リンク（背面）よりこの行を前面に出して、ユーザー名リンクをクリック可能にするためのz-index指定
  ※ z-indexが効くのは position 指定がある要素のみなので position-relative をセット

- if XXX.user.present?

  関連が nil（退会・不整合など）でも落ちないように防御的に分岐させる<br>
  無い場合は "Unknown" 表示にフォールバック

- `<strong> ... </strong>`

  著者名を視覚的に強調。情報設計的にも妥当

- link_to XXX.user.name, user_path(XXX.user), class: "text-decoration-none"

  ユーザー名をプロフィール（/users/:id）へのリンクにする<br>
  text-decoration-none で下線を消してデザインに馴染ませる

- ・ <%= l(XXX.created_at, format: :short) rescue XXX.created_at.to_s %>

  投稿の作成日時。l は I18n のローカライズヘルパー<br>
  format: :short は config/locales/ja.yml の日時フォーマットに従う<br>
  rescue ... は一時的な保険（フォーマット未定義でも生文字で表示）

### それぞれの違い（post / @post / item）

- 一覧（posts/index.html.erb）

  ループ変数が post
  → post.user.name / user_path(post.user) / post.created_at を参照する

- 詳細（posts/show.html.erb）

  単体表示なのでインスタンス変数 @post。
  → @post.user.name / user_path(@post.user) / @post.created_atを参照する

- マイページ（users/show.html.erb）

  タイムラインのループ変数が item（自分の投稿もRTも Post として扱う）
  → item.user.name / user_path(item.user) / item.created_atを参照する
  ※ RT帯は別条件（item.kind == 'repost'）で前に表示済み