# リポストの際に確認ダイアログの実装

## ✅ 行ったこと

- リポストの際に確認ダイアログが出るように実装した

## _repost_button.html.erbの編集
Rails の button_to に data: { turbo_confirm: "〜" } を追加するだけで簡単に実装できる

### 修正前
```
<div id="<%= dom_id(post, :repost_button) %>">
  <% if logged_in? %>
    <% if post.reposted_by?(current_user) %>
      <%= button_to post_repost_path(post), method: :delete,
            class: "btn p-0 border-0 bg-transparent text-success fs-4 d-inline-flex align-items-center",
            data: { turbo: true } do %>
        <i class="bi bi-arrow-repeat"></i>
        <span class="ms-1"><%= post.reposts_count %></span>
      <% end %>
    <% else %>
      <%= button_to post_repost_path(post), method: :post,
            class: "btn p-0 border-0 bg-transparent text-muted fs-4 d-inline-flex align-items-center",
            data: { turbo: true } do %>
        <i class="bi bi-arrow-repeat"></i>
        <span class="ms-1"><%= post.reposts_count %></span>
      <% end %>
    <% end %>
  <% else %>
    <%= link_to login_path,
          class: "btn p-0 border-0 bg-transparent text-muted fs-4 d-inline-flex align-items-center" do %>
      <i class="bi bi-arrow-repeat"></i>
      <span class="ms-1"><%= post.reposts_count %></span>
    <% end %>
  <% end %>
</div>
```

### 修正後
```
<div id="<%= dom_id(post, :repost_button) %>">
  <% if logged_in? %>
    <% if post.reposted_by?(current_user) %>
      <%= button_to post_repost_path(post), method: :delete,
            class: "btn p-0 border-0 bg-transparent text-success fs-4 d-inline-flex align-items-center",
            data: { turbo: true, turbo_confirm: "リツイートを取り消しますか？" } do %>     #追加
        <i class="bi bi-arrow-repeat"></i>
        <span class="ms-1"><%= post.reposts_count %></span>
      <% end %>
    <% else %>
      <%= button_to post_repost_path(post), method: :post,
            class: "btn p-0 border-0 bg-transparent text-muted fs-4 d-inline-flex align-items-center",
            data: { turbo: true, turbo_confirm: "リツイートしますか？" } do %>     　　　　#追加
        <i class="bi bi-arrow-repeat"></i>
        <span class="ms-1"><%= post.reposts_count %></span>
      <% end %>
    <% end %>
  <% else %>
    <%= link_to login_path,
          class: "btn p-0 border-0 bg-transparent text-muted fs-4 d-inline-flex align-items-center" do %>
      <i class="bi bi-arrow-repeat"></i>
      <span class="ms-1"><%= post.reposts_count %></span>
    <% end %>
  <% end %>
</div>
```
