# 詳細画面のタイトルにボタンを追加

## ✅ 行ったこと

- 詳細画面のタイトルにボタンを追加して、配置をデザインした

## 該当部分のコード
```
<div class="flex justify-between items-center w-full">
  <div class="text-2xl">問題詳細</div>
  <div class="flex gap-x-2">
    <%= link_to "編集", edit_question_path(@question), class: "btn btn-outline btn-success" %>
    <%= button_to "削除", @question, method: :delete, class: "btn btn-outline btn-error" %>
  </div>
</div>
```

## `<div class="flex justify-between items-center w-full">`
- flex
flex — display: flex;(flex-direction はデフォルトで row)<br>
(https://tailwindcss.com/docs/display#flex)<br>
つまり、flexのみつけると、要素が横並びになる<br>
縦並びにするには、flex-colにする

- `justify-between`
要素が均等に配置される<br>
(https://tailwindcss.com/docs/justify-content#space-between)

## `<div class="flex gap-x-2">`
- gap-x-2
二つのボタンの、水平方向のpaddingを調整<br>
(https://tailwindcss.com/docs/gap#changing-row-and-column-gaps-independently)
