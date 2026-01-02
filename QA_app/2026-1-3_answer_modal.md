# 質問の回答をモーダルで実装

## ✅ 行ったこと

- 質問の回答をモーダルで表示できるようにした

## 実際のコード
```
<div id="questions">
  <% @questions.each do |question| %>
    <div class="my-8">
      <div class="card bg-base-100 shadow p-6 border-2 border-secondary">
        <div class="flex justify-between items-center">
          <h2 class="text-xl font-bold"><%= question.content %></h2>
          <button class="btn" onclick="my_modal_<%= question.id %>.showModal()">回答を見る</button>
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

## `<button class-"btn onclick~">~~~</dialog>`
- 参考にしたサイト（https://daisyui.com/components/modal/）のDialog modal with a close button at cornerから実装
- IDの部分は、questionのIDを利用した

## `<div class="flex justify-between items-center">`
質問文と、回答ボタンを左端と右端で表示したいので、記述()

## 完成図
[![Image from Gyazo](https://i.gyazo.com/8a09bd45e52782ca512dbce6623004b4.png)](https://gyazo.com/8a09bd45e52782ca512dbce6623004b4)<br>
[![Image from Gyazo](https://i.gyazo.com/3304e48b089e862327f4fec8259390f0.png)](https://gyazo.com/3304e48b089e862327f4fec8259390f0)
