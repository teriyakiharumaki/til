# いいねボタンのモデル作成時のエラー2

## ✅ 行ったこと

- いいねボタンの変化はするが、いいね数が非同期で変化しない問題の修正をした

## 現在の処理
いいねの作成/取り消しのLikesControllerでの動き

- 1.対象の投稿をメモリに読み込む
```
@post = Post.find(params[:post_id])
```
→ これは Rubyオブジェクト（インメモリの @post） を作る<br>
→ この時点の @post.likes_count は「読み込んだ瞬間の値」

- 2.Like を作成 or 削除
```
current_user.likes.find_or_create_by!(post: @post)   # create側
current_user.likes.where(post: @post).destroy_all     # destroy側
```

ここで重要なのは：
- Like に belongs_to :post, counter_cache: true が付いているため、**Like の create/destroy の“あと”に、DB内の posts.likes_count を自動で +1 / -1 してくれる**（コールバックが走る）
- ただし、**更新されるのは DB 上のレコード。最初に読み込んだ @post（Rubyオブジェクト）の likes_count は自動では変わらない**

- 3.そのまま部分テンプレートを描画
```
render turbo_stream: [
  turbo_stream.replace(dom_id(@post, :likes), partial: "posts/likes", locals: { post: @post }),
  ...
]
```

→ ここで @post.likes_count を参照すると、古い値のまま 表示してしまう

## likes_controllerを修正
```
class LikesController < ApplicationController
  include ActionView::RecordIdentifier
  before_action :authenticate_user!
  before_action :set_post

  def create
    current_user.likes.find_or_create_by!(post: @post)
    @post.reload  # ← これを追加（likes_count を最新に）
    respond_to do |format|
      format.html { redirect_back fallback_location: post_path(@post) }
      format.turbo_stream do
        render turbo_stream: [
          turbo_stream.replace(dom_id(@post, :likes),       partial: "posts/likes",       locals: { post: @post }),
          turbo_stream.replace(dom_id(@post, :like_button),  partial: "posts/like_button", locals: { post: @post })
        ]
      end
    end
  end

  def destroy
    current_user.likes.where(post: @post).destroy_all
    @post.reload  # ← ここにも追加
    respond_to do |format|
      format.html { redirect_back fallback_location: post_path(@post) }
      format.turbo_stream do
        render turbo_stream: [
          turbo_stream.replace(dom_id(@post, :likes),       partial: "posts/likes",       locals: { post: @post }),
          turbo_stream.replace(dom_id(@post, :like_button),  partial: "posts/like_button", locals: { post: @post })
        ]
      end
    end
  end

  private
  def set_post
    @post = Post.find(params[:post_id])
  end
end
```

## @post.reload の意味
- DBから最新の状態でもう一度読み直すメソッド
- これにより @post.likes_count が DBで更新済みの値（counter_cache が反映した最新値）に置き換わる
- その最新の @post をパーシャルに渡すので、数字が即時に増減して見える

## 処理の流れ
### 1.ユーザーが「いいね」ボタンを押す

ブラウザから POST /posts/:id/like が送られる。

### 2.Rails がコントローラに到達

- LikesController#create が呼ばれる。
- 中で current_user.likes.find_or_create_by!(post: @post) が実行される。

### 3.ActiveRecord が Like レコードを生成

INSERT INTO likes (user_id, post_id) VALUES (...) が実行される。

### 4.counter_cache の仕組みが働く

- Like モデルには belongs_to :post, counter_cache: true がある。
- これによって ActiveRecord は Like の after_create コールバックの中で、親(Post)の likes_count を更新する SQL を自動発行する。

具体的には 👇
```
UPDATE posts
SET likes_count = COALESCE(likes_count, 0) + 1
WHERE id = <post.id>;
```

### 5.DBの更新が終わる

これで posts.likes_count カラムが最新の値になる。

### 6.コントローラの残りの処理へ

でも @post インスタンスは最初に読み込んだままだから古い値を持っている。<br>
そこで @post.reload を挟むと、もう一度 SELECT が走り、likes_count が更新された値で再読み込みされる。

### 7.ビュー描画 / Turbo Stream で置き換え

_likes.html.erb に渡す post.likes_count が最新値になり、画面に「+1」された状態で表示される。

### 取り消しの場合
流れはほぼ同じで、SQLが逆方向になるだけ
```
UPDATE posts
SET likes_count = COALESCE(likes_count, 0) - 1
WHERE id = <post.id>;
```

## current_user.likes.find_or_create_by!(post: @post)について

### 1.current_user.likes

- これはアソシエーション。
- User が has_many :likes を持っているので、
  current_user.likes は「そのユーザーがしたいいね一覧」を表す ActiveRecord::Relation になる。
- つまり、この時点で WHERE user_id = current_user.id が付いた状態のクエリ準備。

### 2.`.find_or_create_by!(post: @post)`

意味：

- まず探す → likes テーブルから (user_id: current_user.id, post_id: @post.id) に一致するレコードを検索。
- 見つかればそれを返す（DB更新なし）。
- なければ作る → 新しい Like レコードを INSERT して返す。
- ! が付いているので、保存に失敗したら例外を発生させる。

実際のSQL（なかった場合）：
```
SELECT "likes".* FROM "likes" WHERE "likes"."user_id" = 1 AND "likes"."post_id" = 10 LIMIT 1;
INSERT INTO "likes" ("user_id", "post_id", "created_at", "updated_at") VALUES (1, 10, '2025-09-25 12:00:00', '2025-09-25 12:00:00');
UPDATE "posts" SET "likes_count" = COALESCE("likes_count", 0) + 1 WHERE "posts"."id" = 10;
```

### 3.counter_cache の発動

belongs_to :post, counter_cache: true のおかげで、Like の INSERT が成功した直後に ActiveRecord が自動で「親Postの likes_count を +1」する UPDATE を発行する。<br>
つまり、この1行で「Like作成 + Post.likes_count更新」まで完結している。

### この行でDBの更新が行われたので、その次の行の@post.reload で更新を行えばOKになる