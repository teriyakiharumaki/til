# マイページの編集実装

## ✅ 行ったこと

- マイページ編集画面の実装をした

## 1.カラムの追加
ユーザーにbioカラムがないので追加する<br>
*bio→bioは、ソーシャルメディアプラットフォームで、ユーザーが自己紹介や経歴情報を短くまとめたセクションを指します。たとえば、X（旧：Twitter）やInstagramなどのプロフィールページで、「bio」という言葉が使用されます。<br>
(https://kimini.online/blog/archives/52540)
```
rails g migration AddBioToUsers bio:text
rails db:migrate
```

## 2.モデルにバリデーションを追加
app/models/user.rb
```
class User < ApplicationRecord
  has_secure_password
  validates :name,  presence: true
  validates :email, presence: true, uniqueness: true
  validates :bio, length: { maximum: 500 }, allow_blank: true    #追加
  has_many :posts, dependent: :destroy
  has_many :likes, dependent: :destroy
end
```

## 3.ルーティング追加
config/routes.rb
```
  resources :users, only: [:new, :create, :show, :edit, :update]   #edit,update追加
```

## 4.コントローラ
app/controllers/users_controller.rb
```
class UsersController < ApplicationController
  before_action :authenticate_user!, only: [:edit, :update]
  before_action :set_user,      only: [:show, :edit, :update]
  before_action :authorize_me!, only: [:edit, :update]

  ~~~~~~~~~~

  def show
    @posts = @user.posts.includes(:user).order(created_at: :desc)
  end

  def edit; end

  def update
    if @user.update(user_params)
      redirect_to user_path(@user), notice: "プロフィールを更新しました"
    else
      flash.now[:alert] = "更新に失敗しました"
      render :edit, status: :unprocessable_entity
    end
  end

  private

  def user_params
    params.require(:user).permit(:name, :email, :password, :password_confirmation, :bio)   #bio追加
  end

  def set_user
    @user = User.find(params[:id])
  end

  def authorize_me!
    redirect_to user_path(@user), alert: "権限がありません" unless current_user&.id == @user.id
  end
end

```

## 5.ビュー
app/views/users/edit.html.erb
```
<div class="container mt-5" style="max-width: 640px;">
  <h1 class="h4 mb-4">プロフィール編集</h1>

  <%= form_with model: @user do |f| %>
    <div class="mb-3">
      <%= f.label :name, "名前", class: "form-label" %>
      <%= f.text_field :name, class: "form-control", required: true %>
    </div>

    <div class="mb-3">
      <%= f.label :bio, "自己紹介", class: "form-label" %>
      <%= f.text_area :bio, rows: 5, class: "form-control", placeholder: "はじめまして！" %>
      <div class="form-text">※ 500文字までがおすすめ</div>
    </div>

    <%= f.submit "更新する", class: "btn btn-primary" %>
    <%= link_to "戻る", user_path(@user), class: "btn btn-outline-secondary ms-2" %>
  <% end %>
</div>
```

## 6.マイページで表示
app/views/users/show.html.erb のヘッダ付近
```
<h1 class="h4 mb-1"><%= @user.name %></h1>
<% if @user.bio.present? %>
  <div class="mb-3 text-muted"><%= simple_format(h(@user.bio)) %></div>
<% end %>
```

## 7.マイページにリンクを追加
```
    <div class="d-inline-flex gap-2">
      <% if logged_in? && current_user.id == @user.id %>
        <%= link_to "プロフィール編集", edit_user_path(current_user), class: "btn btn-outline-secondary" %>   #リンクを追加
        <%= link_to "つぶやく", new_post_path, class: "btn btn-outline-primary" %>
      <% end %>
    </div>
```