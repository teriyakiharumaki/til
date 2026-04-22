# tailwindcssとdaisyUIの導入

## ✅ 行ったこと

- tailwindcssの導入
- daisyUIの導入

## cdnで導入
game_log/resources/views/layouts/app.blade.php
```
<!doctype html>
<html lang="ja" data-theme="retro">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3.x.x/dist/cdn.min.js"></script>
    <link href="https://cdn.jsdelivr.net/npm/daisyui@5" rel="stylesheet" type="text/css" />                   #ここから
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <link href="https://cdn.jsdelivr.net/npm/daisyui@5/themes.css" rel="stylesheet" type="text/css" />        #ここまで
    <title>@yield('title', 'Game Log')</title>
  </head>
```

参考にした公式サイト(https://daisyui.com/docs/cdn/)

最初の2文で実装可能だが、テーマを利用するため3文目を追加

## トップページのUIデザインの変更
game_log/resources/views/home.blade.php
```
@section('title', 'トップページ')

@section('content')
  <div class="hero min-h-screen">
    <div class="hero-content text-center">
      <div class="max-w-md">
        <h1 class="text-5xl font-bold">GAME LOG</h1>
        <p class="py-6">
          ゲームのレビューを記録するアプリです。
        </p>
        <a href="{{ route('games.index') }}" class="btn btn-primary">
          ゲーム一覧
        </a>
      </div>
    </div>
  </div>
@endsection
```

heroで文字、ボタンがページの真ん中に表示されるように利用<br>
(https://daisyui.com/components/hero/)

一覧へのリンクはボタンで実装<br>
（https://daisyui.com/components/button/）