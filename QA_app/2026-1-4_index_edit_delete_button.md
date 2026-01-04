# 一覧画面で作成と削除機能追加

## ✅ 行ったこと

- 一覧画面で作成と削除ボタンを各投稿に追加した

## 実際のコード
```
<div id="questions">
  <% @questions.each do |question| %>
    <div class="my-8">
      <div class="card bg-base-100 shadow p-6 border-2 border-secondary">
        <div class="flex justify-between items-center">
          <h2 class="text-xl font-bold"><%= question.content %></h2>
          *ここから
          <div class="flex gap-x-2">
            <button class="btn" onclick="my_modal_<%= question.id %>.showModal()">回答を見る</button>
            <%= link_to edit_question_path(question), class: "btn btn-outline btn-success" do %>
              <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
                <path stroke-linecap="round" stroke-linejoin="round" d="m16.862 4.487 1.687-1.688a1.875 1.875 0 1 1 2.652 2.652L10.582 16.07a4.5 4.5 0 0 1-1.897 1.13L6 18l.8-2.685a4.5 4.5 0 0 1 1.13-1.897l8.932-8.931Zm0 0L19.5 7.125M18 14v4.75A2.25 2.25 0 0 1 15.75 21H5.25A2.25 2.25 0 0 1 3 18.75V8.25A2.25 2.25 0 0 1 5.25 6H10" />
              </svg>
            <% end %>
            <%= button_to question, method: :delete, class: "btn btn-outline btn-error", data: { turbo_confirm: "削除しますか？" } do %>
              <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
                <path stroke-linecap="round" stroke-linejoin="round" d="m14.74 9-.346 9m-4.788 0L9.26 9m9.968-3.21c.342.052.682.107 1.022.166m-1.022-.165L18.16 19.673a2.25 2.25 0 0 1-2.244 2.077H8.084a2.25 2.25 0 0 1-2.244-2.077L4.772 5.79m14.456 0a48.108 48.108 0 0 0-3.478-.397m-12 .562c.34-.059.68-.114 1.022-.165m0 0a48.11 48.11 0 0 1 3.478-.397m7.5 0v-.916c0-1.18-.91-2.164-2.09-2.201a51.964 51.964 0 0 0-3.32 0c-1.18.037-2.09 1.022-2.09 2.201v.916m7.5 0a48.667 48.667 0 0 0-7.5 0" />
              </svg>            
            <% end %>
          </div>
          *ここまで
        </div>
        <dialog id="my_modal_<%= question.id %>" class="modal">
          <div class="modal-box">
            <form method="dialog">
              <button class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2">✕</button>
            </form>
            <h3 class="text-lg font-bold">回答</h3>
            <%= question.answer %>
          </div>
        </dialog>
      </div>
    </div>
  <% end %>
</div>
```

## `<div class="flex gap-x-2">`
横並びと、ボタン間の隙間の調整

## `<%= link_to ~ do %>` `<%= button_to ~ do %>`
この形にすることで、do以降の内容がクリックできるようになる

## `<svg>~</svg>`
アイコンをサイト（https://heroicons.com/）から持ってきた

## `data: { turbo_confirm: "削除しますか？" } `
確認のダイアログを追加

## 完成図
[![Image from Gyazo](https://i.gyazo.com/b3fc89ec259c51e5bb828ecd90b18037.png)](https://gyazo.com/b3fc89ec259c51e5bb828ecd90b18037)
