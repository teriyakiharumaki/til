# ログインのルーティングの見直し

## ✅ 行ったこと

- ログインのルーティングについて気になることがあったので修正

## 8/31に学んだこと

resources :users, only: [:new, :create] と書くと、Railsが「RESTfulルーティング」という決まりに従ってルートを自動生成してくれる。(8/31)<br>
ここで、ログインのルーティングは以下で実装している
```
resource :session, only: [:new, :create, :destroy] 
get "login", to: "sessions#new" 
post "login", to: "sessions#create" 
delete "logout", to: "sessions#destroy"
```
これだと、resource :session, only: [:new, :create, :destroy] 要らないのでは？と考えた

## 修正

resource :session, only: [:new, :create, :destroy]を削除

## resource :session, only: [:new, :create, :destroy]で生成されるルート

- GET /session/new → sessions#new (new_session_path)
- POST /session → sessions#create (session_path)
- DELETE /session → sessions#destroy (session_path)

この場合のビューは、
```
<%= link_to "Log in", new_session_path %>
<%= form_with url: session_path, method: :post do %> ... <% end %>
<%= button_to "Logout", session_path, method: :delete %>
```
このようになり、URL が /login / /logout じゃないのでわかりづらい

## get "login", to: "sessions#new" 〜のみの場合

ビューは、
```
<%= link_to "Log in", login_path %>
<%= form_with url: login_path, method: :post do %> ... <% end %>
<%= button_to "Logout", logout_path, method: :delete %>
```
のようになり、URL が直感的でわかりやすくなる

## 結論
```
resource :session, only: [:new, :create, :destroy]
```
と
```
get "login", to: "sessions#new" 
post "login", to: "sessions#create" 
delete "logout", to: "sessions#destroy"
```
は同様のルーティングであり、二つ記述するのは冗長なのでどちらかは消すべき