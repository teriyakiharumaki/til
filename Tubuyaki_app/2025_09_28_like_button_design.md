# いいねボタンのデザインを変更

## ✅ 行ったこと

- いいねボタンのデザインを変更した

## いいねボタンをハートマークになるように実装

```posts/_likes_button_html.erb
<div id="<%= dom_id(post, :like_button) %>">
  <% if logged_in? %>
    <% if post.liked_by?(current_user) %>
      <%= button_to post_like_path(post), method: :delete,
            class: "btn p-0 border-0 bg-transparent text-danger fs-4 d-inline-flex align-items-center",
            data: { turbo: true } do %>
        <i class="bi bi-heart-fill"></i>
        <span class="ms-1"><%= post.likes_count %></span>
      <% end %>
    <% else %>
      <%= button_to post_like_path(post), method: :post,
            class: "btn p-0 border-0 bg-transparent text-muted fs-4 d-inline-flex align-items-center",
            data: { turbo: true } do %>
        <i class="bi bi-heart"></i>
        <span class="ms-1"><%= post.likes_count %></span>
      <% end %>
    <% end %>
  <% else %>
    <%= link_to login_path,
          class: "btn p-0 border-0 bg-transparent text-muted fs-4 d-inline-flex align-items-center" do %>
      <i class="bi bi-heart"></i>
      <span class="ms-1"><%= post.likes_count %></span>
    <% end %>
  <% end %>
</div>
```

_likes.htmlが無駄だったので、_likes_button_html.erbにまとめた
```
<span class="ms-1"><%= post.likes_count %></span>　#この部分
```
それに伴い、_likes.htmlは削除<br>
さらに、エラーが起こるのでlikes_controller.rbを修正
```
turbo_stream.replace(dom_id(@post, :likes),       partial: "posts/likes",       locals: { post: @post }), #この行を削除
```
show.html.erbからも以下の部分を削除（パーシャルファイルがなくなるため）
```
<%= render "posts/likes", post: @post %>
```

 ## 実際の画面

[![Image from Gyazo](https://i.gyazo.com/ca3e0d7b48aed82651f2b2c17ab3f54a.png)](https://gyazo.com/ca3e0d7b48aed82651f2b2c17ab3f54a)

## `_likes_button_html.erb`の各部分解説

### 1. `<div id="<%= dom_id(post, :like_button) %>">`

dom_id は Rails が提供するヘルパー<br>
post オブジェクトと :like_button を渡すと、ユニークなDOM IDが生成される。
- 例: post_12_like_button
Turbo Streamで「部分更新」するとき、このIDがターゲットになる。<br>

### 2. if logged_in?

ログイン状態によって表示を分ける。
- ログイン済みなら「いいね」「取り消し」ボタンを表示
- 未ログインならログインページへのリンクを表示

### 3. if post.liked_by?(current_user)

カスタムメソッド（Postモデルに実装済み）<br>
この投稿に対して「今のユーザーがいいねしてるか？」を判定する。<br>
trueなら「取り消し」ボタン、falseなら「いいね」ボタンを表示。

### 4. button_to post_like_path(post), method: :post / :delete

button_to はフォーム付きの `<button>` を生成するRailsヘルパー<br>
post_like_path(post) はルーティングで定義された「いいね作成/削除」のエンドポイント。
- method: :post → いいねする
- method: :delete → いいねを取り消す

### 5. class="btn p-0 border-0 bg-transparent ..."

Bootstrapのユーティリティクラスを組み合わせて「Twitter風のアイコンボタン」にしている。
- btn → ボタンの基本スタイル
- p-0 → paddingを0にして余白を消す
- border-0 → 枠線を消す
- bg-transparent → 背景色を透明にする
- text-danger → 赤色（いいね済み）
- text-muted → グレー（未いいね）
- fs-4 → アイコンとテキストを大きめに
- d-inline-flex align-items-center → アイコンと数字を横並び＆縦中央揃え

### 6. `<i class="bi bi-heart"></i> / <i class="bi bi-heart-fill"></i>`

Bootstrap Iconsのクラス
- bi-heart → アウトラインのハート（未いいね状態）
- bi-heart-fill → 塗りつぶしハート（いいね済み状態）

### 7. `<span class="ms-1"><%= post.likes_count %></span>`

- post.likes_count → 投稿に紐づく「いいね数」を表示。
- ms-1 → margin-start 1（左に少し余白）をつけて、ハートと数字の間に隙間を作る。

### 8. data: { turbo: true }

- Turbo Streamsを有効にして、非同期で部分更新する。
- 押すたびに「ボタンと数字の部分だけ」差し替えられる。

## 動作の流れ
1. ユーザーがハートを押す
2. Railsが LikesController#create または destroy を実行
3. その中で Turbo Stream レスポンスを返す
4. DOM ID post_◯_like_button をターゲットに置き換え → アイコンと数字が切り替わる