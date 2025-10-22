# 投稿の削除機能の仕組み

## ✅ 行ったこと

- つぶやいた投稿を削除する機能の仕組みについての解説を考えた

## ざっくりとした流れ

### ①ユーザー操作
一覧カードの「削除」ボタン（button_to ... method: :delete）をクリックする

### ②Turboによる送信
button_to はデフォで Turbo を使って DELETE /posts/:id を非同期送信する
サーバへは通常のフォーム送信と同じ CSRF 安全性で飛ぶ

### ③コントローラがDELETEを処理
PostsController#destroy が呼ばれ、対象の @post を削除

### ④Turbo Stream レスポンスを返す
コントローラは format.turbo_stream を選び、Turbo Stream テンプレート（destroy.turbo_stream.erb）を返す
このレスポンスのContent-Typeは text/vnd.turbo-stream.html

### ⑤TurboがDOMに適用
レスポンスの`<turbo-stream action="remove" target="post_123">`を受け取った Turbo が
id="post_123" の要素をDOMから削除する（JS不要）

### ⑥結果：リロードなしでカードが消える
他のカードはそのまま。画面は一瞬で更新完了

## ✅ 具体的なコード

### 一覧カードに一意なIDを振る（= “消される側”）
```
<!-- app/views/posts/index.html.erb -->
<% @posts.each do |post| %>
  <div id="<%= dom_id(post) %>" class="card mb-3 position-relative">            #ココ！
    <div class="card-body position-relative">

      <div class="mt-2 pt-2 border-top position-relative z-3 d-flex align-items-center gap-3">
        <%= render "posts/like_button", post: post %>

        <% if logged_in? && post.user_id == current_user.id %>
          <%= button_to "削除",
                post_path(post),
                method: :delete,
                data: { turbo_confirm: "この投稿を削除します。よろしいですか？" },
                class: "btn btn-outline-danger btn-sm" %>
        <% end %>

        <%= link_to "", post_path(post), class: "stretched-link" %>
      </div>
    </div>
  </div>
<% end %>
```
- dom_id(post) は Rails のヘルパーで、通常 "post_#{post.id}" を返す<br>
- これが Turbo Stream の target と一致することで、該当のカードだけ消せる

### コントローラ：destroyでTurbo応答
```
# app/controllers/posts_controller.rb
class PostsController < ApplicationController
  before_action :authenticate_user!, only: [:new, :create, :destroy]
  before_action :set_post, only: [:show, :destroy]
  before_action :authorize_owner!, only: [:destroy]

  def destroy
    @post.destroy
    respond_to do |format|
      format.html { redirect_to posts_path, notice: "投稿を削除しました" }     # フォールバック
      format.turbo_stream                                                   # 次のテンプレに委譲
    end
  end

  private
  def set_post; @post = Post.find(params[:id]); end
  def authorize_owner!; redirect_to posts_path, alert: "権限がありません" unless @post.user_id == current_user&.id; end
end
```
Turbo環境以外でも壊れないように、HTMLレスポンスも用意しておくのが定石

### Turbo Stream テンプレート：DOMから削除
```
<!-- app/views/posts/destroy.turbo_stream.erb -->
<%= turbo_stream.remove dom_id(@post) %>
```
これが出力すつのは以下のようなHTML
```
<turbo-stream action="remove" target="post_123"></turbo-stream>
```
ブラウザ側の Turbo がこれを解釈し、id="post_123" の要素をremove(削除)

## posts_controlerのturboの処理については以下の過去ログを参照
2025_09_20_like_button_detail_posts_controller.md

## ✅ もうすこしturboのcontorllerでの処理を詳しく調べた

### 1.なぜ format.turbo_stream が選ばれるのか？

button_to（Rails 7 / Turbo有効）で削除ボタンを押す<br>
→ ブラウザは XHR（fetch） で DELETE /posts/:id を送る<br>

このとき Turbo は Accept ヘッダに
```
text/vnd.turbo-stream.html, text/html, application/xhtml+xml
```
などを含む<br>

Rails の respond_to はこの Accept を見て、format.turbo_stream を選択<br>
サーバーログに Processing by PostsController#destroy as TURBO_STREAM と出るのはこのため

Turboが無効・非対応なら Accept: text/html で来るので、format.html が選ばれます（フォールバック）

### 2.format.turbo_stream が選ばれると何が起きるのか
```
def destroy
  @post.destroy
  respond_to do |format|
    format.html { redirect_to posts_path, notice: "投稿を削除しました" }
    format.turbo_stream  # ← これだけ書くと Rails がテンプレートを探す
  end
end
```
Rails は app/views/posts/destroy.turbo_stream.erb を探してレンダリング<br>
このテンプレートの中で、例えば：
```
<%= turbo_stream.remove dom_id(@post) %>
```
を返すと、実際のレスポンス本文は以下
```
<turbo-stream action="remove" target="post_123"></turbo-stream>
```

### 3.レスポンスを受け取ったブラウザ側
- Turbo（クライアントJS）が text/vnd.turbo-stream.html を見て、HTMLを“DOM操作命令”として解釈
- `<turbo-stream action="remove" target="post_123">`<br>
  → ドキュメント内の id="post_123" の要素を remove() で消す
- ここで重要なのは：<br>
  DOMの操作はすべてクライアント側（Turbo JS）が行う<br>
  サーバ側は「どのIDをどうしろ」という宣言（命令）を返すだけ

### 4.DB と 画面のタイミング
- destroy アクションの先頭で DBからはすでに @post.destroy 済み
- その後、画面（DOM）を消すための命令（Turbo Stream）を返す
- destroyの処理が行われる <br>
  → format.turbo_stream で destroy.turbo_stream.erb を処理 <br>
  → dom_id(@post) を target にした `<turbo-stream action="remove">` を返す <br>
  → ブラウザ側でそのIDの要素が非同期にDOMから消える<br>
  という流れ

### 5.dom_id(@post) が効く理由
- dom_id(record) は Rails のヘルパーで、基本的に "#{model_name}_#{id}"（例：post_123）を返す
- 一覧のカード側で
  ```
  <div id="<%= dom_id(post) %>"> ... </div>
  ```
  と出しておけば、サーバ（destroy側）とクライアント（一覧DOM）のIDが一致
- これにより 「どの要素を消すか」がズレずに指示できる

## ✅ Acceptヘッダとは？
HTTPリクエストの一部で、クライアント（ブラウザ）が<br>
「自分はこの形式のレスポンスを受け取れるよ」<br>
とサーバに伝える仕組み

### 例：普通のHTMLリクエスト
ブラウザがページ遷移するとき、こんなヘッダを送る：
```
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
```
これは「HTMLでもXHTMLでもXMLでも大丈夫。最悪は任意の形式でもいいよ」という意味

### 例：Turbo付きフォーム送信
Rails 7 + Turbo でフォームや button_to を使うと、非同期でリクエストが送られる<br>
そのときは：
```
Accept: text/vnd.turbo-stream.html, text/html, application/xhtml+xml
```
というヘッダになります
ここで text/vnd.turbo-stream.html がミソ

- 「もしサーバーが Turbo Streams を返せるなら、その形式を最優先でほしい」
- Rails 側はこれを見て format.turbo_stream を選ぶ

### Rails側での処理

コントローラの respond_to は、リクエストの Accept を確認して分岐
```
respond_to do |format|
  format.html         # Accept に text/html があれば
  format.turbo_stream # Accept に text/vnd.turbo-stream.html があれば
end
```

だからサーバーログに<br>
Processing by PostsController#destroy as TURBO_STREAM<br>
と出る

### Acceptまとめ

- Acceptヘッダ = 「このフォーマットなら理解できます」とブラウザが宣言するリスト
- Turboはここに text/vnd.turbo-stream.html を入れてくる
- Railsはそれを見て format.turbo_stream を実行
- だから「Turbo対応なら非同期で画面だけ更新」「非対応なら普通にHTMLでリロード」という分岐が自然にできる

## Accept: text/vnd.turbo-stream.html, text/html, application/xhtml+xmlについて
これはブラウザがサーバーに「次の順で返してくれると助かるよ」と伝えている
- text/vnd.turbo-stream.html ← 最優先
→ もし返せるなら Turbo Stream 形式を返してほしい
- text/html
→ もし Turbo Stream が無理なら普通の HTML を返してほしい
- application/xhtml+xml
→ それもダメなら XHTML でもいい

### 優先度の仕組み
- リストの左から順番に「できればこれで」という希望を出している
- もっと細かく言うと、q パラメータ（quality factor）が付くと優先度を数値で指定できる：
```
Accept: text/html;q=0.9, application/json;q=0.8, */*;q=0.5
```
→ text/html を優先、次に application/json、最悪は何でもOK