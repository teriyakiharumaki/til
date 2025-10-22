# フラッシュメッセージの実装について③

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
<div class="alert alert-<%= level %> alert-dismissible fade show" role="alert">
```
```
<button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
```

## 解説１

### 1. `<div class="alert alert-<%= level %> ...">`
- alert → Bootstrapが「アラート用デザイン」と認識するために必須のクラス
- alert-<%= level %> → level に代入した文字列がここに差し込まれる。
例：
```
<div class="alert alert-success ...">
```
```
<div class="alert alert-danger ...">
```
つまり flash[:notice] なら緑色、flash[:alert] なら赤色のボックスになる。

### 2. alert-dismissible
- 「閉じるボタンをつけられるアラート」にするクラス。
- このクラスがあると `<button class="btn-close">` でアラートを閉じられるようになる。

### 3. fade show
- Bootstrapが用意している「アニメーション効果」のクラス
- fade → フェードイン・フェードアウトできる準備
- show → 今表示中であることを意味する。閉じるときは show が外れてフェードアウト。

### 4. role="alert"
- HTMLのARIA属性（アクセシビリティ対応）
- スクリーンリーダーなど支援技術に「これはアラート（通知メッセージ）ですよ」と伝える役割
- 見た目には関係ないけど、ユーザー補助の観点で大事。

##　解説2

### 1. `<button type="button">`
- type="button" は 「このボタンはフォーム送信用じゃない」 という指定（デフォルトだと type="submit" になってしまうことがあるから、ここで明示してる）

### 2. class="btn-close"
- Bootstrapが用意している「×（バツ印）の見た目」のクラス
- 実際に文字を書かなくても、CSSで「小さな閉じるアイコン」が表示される。
- だから `<span>×</span>` みたいに書く必要がない。

### 3. data-bs-dismiss="alert"
- BootstrapのJavaScriptに「このボタンを押したら、対象の .alert を閉じる動作をしてね」と伝える。
- data-bs-dismiss="alert" がなければ、押しても何も起きない。

### 4. aria-label="Close"
- これは アクセシビリティ対応。
- 見た目は「×」だけど、スクリーンリーダーだと「これは閉じるボタンだ」と分からない。
- aria-label="Close" を付けることで「閉じる」と読み上げられるようになる。
- 視覚的に見えないユーザーにも操作が伝わる。

## 実装で参考になる公式サイト
https://getbootstrap.jp/docs/5.3/components/alerts/