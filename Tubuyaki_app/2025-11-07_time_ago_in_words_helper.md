# 投稿日を「◯分前／◯日前」表示にする

## ✅ 行ったこと

- 投稿日を「◯分前／◯日前」表示になるように、helperで実装した

## 実装
app/helpers/posts_helper.rb
```
module PostsHelper
  def posted_time_ago(post)
    "#{time_ago_in_words(post.created_at)}前"
  end
end
```

## ビューで表示（投稿一覧で、名前の横に表示）
app/views/posts/index.html.erb
```
  <% @posts.each do |post| %>
    <div id="<%= dom_id(post) %>" class="card mb-3 position-relative">
      <div class="card-body position-relative">
        <div class="small text-muted mb-1 position-relative z-3">
          <% if post.user.present? %>
            <strong><%= link_to post.user.name, user_path(post.user), class: "text-decoration-none" %></strong>
          <% else %>
            <strong>Unknown</strong>
          <% end %>
          ・ <%= posted_time_ago(post) %>       #ここに記述
        </div>
```

## 解説
前提：ヘルパーは、ビューでよく使う処理やロジックをまとめておく場所

### module PostsHelper

- Helperは通常、コントローラ名に対応するモジュールとして作られる（PostsController に対応するのが PostsHelper）
- Railsが自動でビューにこのモジュールを include してくれるから、`<%= posted_time_ago(post) %>` のように直接使える。

### def posted_time_ago(post)

- 「投稿の時間を“〜前”形式で返すメソッド」を定義
- 引数として post オブジェクト（1件分の投稿）を受け取る

### time_ago_in_words(post.created_at)

- Railsの ActionView::Helpers::DateHelper に含まれるメソッド
- 時間差を「○分」「○時間」「○日」などの日本語で返す
  - 例：time_ago_in_words(2.hours.ago) → "約2時間"
- 日本語化は rails-i18n が導入されていれば自動で反映される

### "#{ ... }前"
- time_ago_in_words は「約2時間」までしか出さないので、"前" を文字列結合して "約2時間前" の形で表示されるようにする