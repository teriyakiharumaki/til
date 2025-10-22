# いいねボタンのモデル作成時のエラー

## ✅ 行ったこと

- いいねボタンを押した時に意図する挙動が起きていないので修正を行なった

## 問題

いいねを押した時には、いいねを押しました！って出てきてカウントも増えるが、<br>
いいねを取り消すときはフラッシュメッセージが出ずに、カウントが減って表示されない

##　いいねを取り消す際のログ
```
tubuyaki-web-1 | 03:53:35 web.1 | Started DELETE "/posts/4/like" for 172.20.0.1 at 2025-09-17 03:53:35 +0900 
tubuyaki-web-1 | 03:53:35 web.1 | Cannot render console from 172.20.0.1! Allowed networks: 127.0.0.0/127.255.255.255, ::1 
tubuyaki-web-1 | 03:53:35 web.1 | Processing by LikesController#destroy as TURBO_STREAM tubuyaki-web-1 | 03:53:35 web.1 | Parameters: {"authenticity_token"=>"[FILTERED]", "post_id"=>"4"} 
tubuyaki-web-1 | 03:53:35 web.1 | User Load (2.0ms) SELECT "users".* FROM "users" WHERE "users"."id" = 1 LIMIT 1 /*action='destroy',application='Myapp',controller='likes'*/ 
tubuyaki-web-1 | 03:53:35 web.1 | ↳ app/controllers/application_controller.rb:7:in current_user' 
tubuyaki-web-1 | 03:53:35 web.1 | Post Load (0.7ms) SELECT "posts".* FROM "posts" WHERE "posts"."id" = 4 LIMIT 1 /*action='destroy',application='Myapp',controller='likes'*/ 
tubuyaki-web-1 | 03:53:35 web.1 | ↳ app/controllers/likes_controller.rb:38:in set_post' 
tubuyaki-web-1 | 03:53:35 web.1 | Like Load (0.9ms) SELECT "likes".* FROM "likes" WHERE "likes"."user_id" = 1 AND "likes"."post_id" = 4 /*action='destroy',application='Myapp',controller='likes'*/ 
tubuyaki-web-1 | 03:53:35 web.1 | ↳ app/controllers/likes_controller.rb:22:in destroy' 
tubuyaki-web-1 | 03:53:35 web.1 | TRANSACTION (0.2ms) BEGIN /*action='destroy',application='Myapp',controller='likes'*/ 
tubuyaki-web-1 | 03:53:35 web.1 | ↳ app/controllers/likes_controller.rb:22:in destroy' 
tubuyaki-web-1 | 03:53:35 web.1 | Like Destroy (2.2ms) DELETE FROM "likes" WHERE "likes"."id" = 4 /*action='destroy',application='Myapp',controller='likes'*/ 
tubuyaki-web-1 | 03:53:35 web.1 | ↳ app/controllers/likes_controller.rb:22:in destroy' 
tubuyaki-web-1 | 03:53:35 web.1 | Post Update All (0.8ms) UPDATE "posts" SET "likes_count" = COALESCE("likes_count", 0) - 1 WHERE "posts"."id" = 4 /*action='destroy',application='Myapp',controller='likes'*/ 
tubuyaki-web-1 | 03:53:35 web.1 | ↳ app/controllers/likes_controller.rb:22:in destroy' 
tubuyaki-web-1 | 03:53:35 web.1 | TRANSACTION (0.8ms) COMMIT /*action='destroy',application='Myapp',controller='likes'*/ 
tubuyaki-web-1 | 03:53:35 web.1 | ↳ app/controllers/likes_controller.rb:22:in destroy' 
tubuyaki-web-1 | 03:53:35 web.1 | Rendered posts/_likes.html.erb (Duration: 0.0ms | GC: 0.0ms) 
tubuyaki-web-1 | 03:53:35 web.1 | Like Exists? (0.7ms) SELECT 1 AS one FROM "likes" WHERE "likes"."post_id" = 4 AND "likes"."user_id" = 1 LIMIT 1 /*action='destroy',application='Myapp',controller='likes'*/ 
tubuyaki-web-1 | 03:53:35 web.1 | ↳ app/models/post.rb:10:in liked_by?' 
tubuyaki-web-1 | 03:53:35 web.1 | Rendered posts/_like_button.html.erb (Duration: 1.9ms | GC: 0.0ms) tubuyaki-web-1 | 03:53:35 web.1 | Rendered shared/_flash.html.erb (Duration: 0.0ms | GC: 0.0ms) 
tubuyaki-web-1 | 03:53:35 web.1 | Completed 200 OK in 84ms (Views: 0.2ms | ActiveRecord: 8.2ms (6 queries, 0 cached) | GC: 1.4ms) 
tubuyaki-web-1 | 03:53:35 web.1 | 
tubuyaki-web-1 | 03:53:35 web.1 |
```

## フラッシュメッセージについて解決
いいねを押したり取り消したりするたびにフラッシュメッセージを表示するのは目障りになると思い、いいね関係の処理にフラッシュメッセージを表示しないようにすることで解決

- ①コントローラーをスリム化
```
# app/controllers/likes_controller.rb
class LikesController < ApplicationController
  include ActionView::RecordIdentifier
  before_action :authenticate_user!
  before_action :set_post

  def create
    current_user.likes.find_or_create_by!(post: @post)
    respond_to do |format|
      format.html { redirect_back fallback_location: post_path(@post) }   # notice削除
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
    respond_to do |format|
      format.html { redirect_back fallback_location: post_path(@post) }   # notice削除
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

- ②レイアウト側はそのまま
shared/flash は他の画面（ログイン/投稿など）で使うので残す

これでいいねをした時の挙動はボタンが「いいね」と「いいねを取り消す」の遷移と、数字が変わることのみになったはず

## 残った問題
ボタンの変化はするが、いいね数が変化しない
