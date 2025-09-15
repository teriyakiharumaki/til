# フラッシュメッセージの実装について②

## ✅ 行ったこと

- フラッシュメッセージの中身について調べた

## フラッシュメッセージのパーシャルファイル
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

## 今回解説する文
```
<% level = { notice: "success", alert: "danger" }.fetch(key.to_sym, "info") %>
```

## 解説

### 1. flash のキーは文字列やシンボル
コントローラでこう書くと：
```
redirect_to posts_path, notice: "ログインに成功しました！"
```
flash にはこんなデータが入る：
```
{ "notice" => "ログインに成功しました！" }
```
key は "notice"（文字列）<br>
これを .to_sym でシンボルに変換すると :notice になる。

### 2. ハッシュで対応表を用意
```
{ notice: "success", alert: "danger" }
```
これは「Railsのキー → Bootstrapのクラス名」の対応表
- :notice → "success" （緑色のアラート）
- :alert → "danger" （赤色のアラート）

### 3. .fetch で取り出す
```
.fetch(key.to_sym, "info")
```
- key.to_sym が :notice なら "success"
- key.to_sym が :alert なら "danger"
それ以外（例えば :error とか独自のキー）なら "info" を返す（デフォルト値）

### 4. 代入される値
例えば、flash[:notice] = "ログインに成功しました！" の場合：

- key = "notice"
- key.to_sym = :notice
- { notice: "success", alert: "danger" }.fetch(:notice, "info") → "success"

結果、level = "success" が入る。

### 5. 実際のHTMLに組み込まれる
```
<div class="alert alert-<%= level %>">
```
となるので、

- notice の場合 → `<div class="alert alert-success">`
- alert の場合 → `<div class="alert alert-danger">`
- その他の場合 → `<div class="alert alert-info">`

になる

## lebelとは
level という変数に文字列を代入してるだけ<br>
でも代入する時にちょっとした「処理（マッピング）」をしている
```
level = { notice: "success", alert: "danger" }.fetch(key.to_sym, "info")
```
ここでやってることは、
- { notice: "success", alert: "danger" } というハッシュ（対応表）を用意
- key.to_sym に応じた値を取り出す
- 該当がなければ "info" を返す
- それを level 変数に代入