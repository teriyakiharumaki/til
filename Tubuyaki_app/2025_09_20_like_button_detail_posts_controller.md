# いいねボタン解説

## ✅ 行ったこと

- いいねボタンのコントローラーについて解説

## app/controllers/posts_controller.rb
```
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

### 1. アクションの全体像
```
def like
  @post = Post.find(params[:id])                 # 1. どの投稿か特定
  @post.increment!(:likes_count)                 # 2. likes_count を +1 更新
  respond_to do |format|                         # 3. リクエストの形式に応じて処理を分ける
    format.html { ... }                          # 4. 通常リクエスト (HTML)
    format.turbo_stream do                       # 5. Turbo Stream (非同期更新)
      ...
    end
  end
end
```

### 2. @post = Post.find(params[:id])
ルーティングで /posts/:id/like になってるから、:id がパラメータとして送られてくる。<br>
そのIDで対象の投稿をDBから探して @post に格納。

### 3. @post.increment!(:likes_count)
increment! は ActiveRecord の便利メソッド<br>
指定カラムを 1 増やして即DBに保存する
```
@post.likes_count = @post.likes_count + 1
@post.save!
```
と同じ意味（! が付いてるから、保存に失敗したら例外を投げる。）
👉 これで「いいね数 +1」が完了

### 4. respond_to do |format|

リクエストが 普通のHTML なのか、Turbo Streams なのかで処理を分けている

format.html { ... }
- 普通のフォーム送信や非Turboブラウザからのアクセスならこっち。
- redirect_back → 前のページにリダイレクトする。
- fallback_location: post_path(@post) → 戻る先が無ければその投稿の詳細ページに飛ばす。
- notice: "いいねしました" → フラッシュメッセージを設定。

format.turbo_stream do ... end
- Turbo (Rails 7標準で入ってる Hotwire/Turbo) によるリクエストならこっち。
- ページ全体をリロードせず、部分的に更新できる。
- ここでは「いいね数の部分」と「フラッシュメッセージ」を差し替えている。

### 5. turbo_stream.replace
```
render turbo_stream: [
  turbo_stream.replace(
    dom_id(@post, :likes),                       # 置き換え先のID
    partial: "posts/likes",                      # 差し替えるパーシャル
    locals: { post: @post }                      # パーシャルに渡すデータ
  ),
  turbo_stream.replace(
    "flash",                                     # id="flash" の部分を
    partial: "shared/flash"                      # flashパーシャルに差し替え
  )
]
```
1つ目
- dom_id(@post, :likes) → 例: "post_5_likes"
- ビュー側の app/views/posts/_likes.html.erb に対応
- 押した投稿の「いいねカウント部分」だけを最新のHTMLに置き換える。

2つ目
- "flash" → レイアウトにある `<div id="flash">...</div>` の部分。
- そこを shared/_flash.html.erb で再描画する。
- これで Turbo リクエストでもフラッシュメッセージが表示される。

## ❓疑問：結局format.turbo_steamが実装されてるならformat.html { ... } の部分っていらないのでは？

結論；両方書くのがベスト<br>

### 1. Turbo Stream が呼ばれるのは「Turbo経由のリクエスト」のときだけ

Rails 7 の button_to はデフォルトで Turboを使った非同期リクエストになるから、たしかに多くの場合 format.turbo_stream 側が実行される。<br>
でもすべてがTurboとは限らない。<br>
例：
- 古いブラウザや Turbo無効化環境（JavaScript切ってるとか）
- API的に curlで POST /posts/:id/like を叩いた場合
- テストコードで post like_post_path(@post) した場合
こういう時は Turbo Stream を返してもブラウザやクライアントは理解できない。<br>
だから フォールバックとして format.html を残しておくのが安心。

### 2. HTMLレスポンスはリダイレクトの役割
```
format.html { redirect_back fallback_location: post_path(@post), notice: "いいねしました" }
```
これがあると、Turboを使っていなくても「いいねボタンを押した後に前のページへ戻ってフラッシュを表示する」動きになる。<br>
つまり Turboが効かない環境でも最低限アプリが動作する保証になる。