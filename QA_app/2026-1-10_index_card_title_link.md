# カードのタイトルをリンク化

## ✅ 行ったこと

- 一覧画面にある各問題のタイトルをリンク化し、詳細画面に飛ぶように実装した

## 実際のコード
index.html.erb
```
<div id="questions">
  <% @questions.each do |question| %>
    <div class="my-8">
      <div class="card bg-base-100 shadow p-6 border-2 border-secondary">
        <div class="flex justify-between items-center">
          #ここから
          <%= link_to question, class: "text-xl font-bold link link-hover" do %>
            <%= question.content %>
          <% end %>
          #ここまで
          <div class="flex gap-x-2">
            ~~~~
```

## 参考にしたサイトの部分
https://daisyui.com/components/link/#show-underline-only-on-hover

## つまった部分
link_toのテキストの部分を<%= question.content %>にする方法がわからなかった<br>

解説してたサイト→https://qiita.com/wangqijiangjun/items/df193c92b7c948c46fbd<br>

よく確認したら、アイコンで編集削除ボタンを実装した際に利用していた
