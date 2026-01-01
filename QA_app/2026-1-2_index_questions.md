# 一覧画面の質問のデザイン

## ✅ 行ったこと

- 一覧画面の質問をカードでデザインした 

## 実際のコード
```
<div id="questions">
  <% @questions.each do |question| %>
    <div class="my-8">
      <div class="card bg-base-100 shadow p-6 space-y-4 border-2 border-secondary">
        <h2 class="text-xl font-bold"><%= question.content %></h2>
        <p class="text-base">
          <span class="font-semibold">回答：</span>
          <%= question.answer %>
        </p>
      </div>
    </div>
  <% end %>
</div>
```

## 質問一つ一つをカード化して表示する
参考にしたところ（https://daisyui.com/components/card/）

## `<div class="my-8">`
カード間のスペースを決めている<br>
(https://tailwindcss.com/docs/margin#adding-vertical-margin)

## `<div class="card bg-base-100 shadow p-6 space-y-4 border-2 border-secondary">`
- `p-6 space-y-4`
これでカードの中身を作っている

- `border-2 border-secondary`
card-borderでも実装できたが、色が薄かったのでこれで実装
