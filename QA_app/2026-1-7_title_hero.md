# タイトル画面のデザインを変更

## ✅ 行ったこと

- タイトル画面のデザインを変更した

## 実際のコード
```
<div class="hero">
  <div class="hero-content text-center">
    <div class="max-w-md">
      <h1 class="text-5xl font-bold py-6">ようこそ！</h1>
      <h1 class="text-5xl font-bold">一問一答アプリへ！</h1>
      <p class="py-6">
        一問一答ができるアプリです
      </p>
      <%= link_to "質問一覧へ", questions_path, class: "btn btn-primary" %>
    </div>
  </div>
</div>
```

## 参考にしたサイトの部分
https://daisyui.com/components/hero/

## 変更した部分

### `<div class="hero">`
bg-base-200 min-h-screenの削除<br>
背景色は指定しないで良く、余白も要らない

### `<h1 class="text-5xl font-bold py-6">ようこそ！</h1>`
二行にするため、py-6を追加

## 実際の画面
[![Image from Gyazo](https://i.gyazo.com/07940b108280b7f40c338e794492de6e.png)](https://gyazo.com/07940b108280b7f40c338e794492de6e)
