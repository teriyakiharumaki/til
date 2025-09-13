# フラッシュメッセージの実装について

## ✅ 行ったこと

- フラッシュメッセージを実装していたと思っていたが、表示されていなかったので修正した

## 実装
 
### フラッシュメッセージを表示するために、パーシャルファイルで実装
app/views/shared/_flash.html.erb
```
<% flash.each do |key, message| %>
  <% level = { notice: "success", alert: "danger" }.fetch(key.to_sym, "info") %>
  <div class="alert alert-<%= level %> alert-dismissible fade show" role="alert">
    <%= message %>
    <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
  </div>
<% end %>
```
このファイルを、application.html.erb のレイアウトファイルに以下のように書くことで実装
```
  <body>
    <%= render "shared/navbar" %>

    <div class="container mt-3" id="flash">
      <%= render "shared/flash" %>
    </div>

    <%= yield %>
  </body>
```

### パーシャルの中身を解説

1.flash.each
- 画面に渡されたフラッシュメッセージ（notice, alert など）を1つずつ取り出す。
- key → notice や alert の種類
- message → 実際のメッセージ文字列

2.Bootstrap用のクラス変換
```
level = { notice: "success", alert: "danger" }.fetch(key.to_sym, "info")
```
- Railsのflash[:notice] を Bootstrap の alert-success に変換
- Railsのflash[:alert] を Bootstrap の alert-danger に変換
- その他（未定義のkey）は alert-info として扱う

3.Bootstrapのアラートコンポーネント
```
<div class="alert alert-<%= level %> alert-dismissible fade show">
  ...
</div>
```
- alert-success（緑）, alert-danger（赤）, alert-info（青）といった見た目に変換
- alert-dismissible fade show → 閉じるボタン付きでアニメーション表示

4.閉じるボタン
```
<button type="button" class="btn-close" data-bs-dismiss="alert"></button>
```
Bootstrapの機能で、クリックするとフラッシュが消えるようになる

### flash.eachについて
<% flash.each do |key, message| %> の「取り出す」とは<br>
→flash というハッシュ形式のデータ構造から、中身を1つずつ取り出して処理すること

具体的な説明<br>
Railsの flash は、実際にはこんなハッシュっぽいデータを持ってることが多い：
```
flash[:notice] = "投稿が成功しました"
flash[:alert]  = "エラーが発生しました"
```
この時の flash の中身イメージはこう：
```
{
  "notice" => "投稿が成功しました",
  "alert"  => "エラーが発生しました"
}
```
flash.each の動き→Rubyの each はハッシュに対して使うと「キーと値のペア」を順番に取り出す。
```
flash.each do |key, message|
  # 1回目: key = "notice", message = "投稿が成功しました"
  # 2回目: key = "alert",  message = "エラーが発生しました"
end
```
だから、このループの中では key が種類 (notice/alert)、message が実際の文字列メッセージである<br>

「取り出す」を噛み砕くと

- flash は「通知の入れ物（ハッシュ）」
- each は「入れ物の中を順番にのぞいて、キーと値を取り出す処理」
- do |key, message| ... end のブロックの中で、取り出したデータを使ってHTMLを作っている

つまり「flash に入っているすべてのフラッシュメッセージを1つずつ取り出して画面に表示する」処理になっている

