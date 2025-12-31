# 一覧画面に罫線を追加

## ✅ 行ったこと

- 一覧画面に罫線の追加を行なった

## 該当部分のコード
views/questions/index.html.erb
```
<div id="questions">
  <% @questions.each do |question| %>
    <div class="flex w-full flex-col">
      <div class="divider divider-neutral"></div>
    </div>
    <%= render question %>
    <p>
      <%= link_to "詳細", question %>
    </p>
    <% end %>
</div>

<div class="flex w-full flex-col">
  <div class="divider divider-neutral"></div>
</div>

<%= link_to "新しい問題を作る", new_question_path %>

```

## 罫線部分のみ
```
<div class="flex w-full flex-col">
  <div class="divider divider-neutral"></div>
</div>
```
参考サイトの該当部分（Divider with colors）<br>
flex flex-colは複数の要素を縦並びにするためのものなので、罫線一個のみなら要らないので削除
```
<div class="w-full">
  <div class="divider divider-neutral"></div>
</div>
```

## 編集、作成、詳細画面のタイトルで罫線を実装する際は、タイトルと縦並びにするのでまとめる
```
<div class="flex flex-col w-full">
  <div class="text-2xl">問題編集</div>
  <div class="divider divider-neutral"></div>
</div>
```

## 参考にしたサイト
daisyUI公式サイト<br>
(https://daisyui.com/components/divider/)
