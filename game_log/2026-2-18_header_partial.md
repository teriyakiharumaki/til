# ヘッダーのパーシャル作成

## ✅ 行ったこと

- ヘッダーのパーシャル作成について

## 作成手順

### ①ヘッダーのパーシャルファイルの作成
resources/views/partials/header.blade.php
```
<header>
  <h1>
    <a href="/">🎮 Game Log</a>
  </h1>
  <hr>
</header>
```

### ②layouts/app.blade.php を修正
```
<body>

  @include('partials.header')

  <main>
    @yield('content')
  </main>

</body>
```

## `@includeと@yield?`
@includeは、**別ファイルの中身をそのままここに読み込む**

### @yieldとの違い？

#### 🔵 @include
→部品を差し込む

例：

- header
- footer
- ナビ
- ボタンのパーツ

固定パーツを読み込む

#### 🟢 @yield
→子ビューに「ここに入れてね」とお願いする場所

レイアウト側（親）：
```
<main>
  @yield('content')
</main>
```

子ビュー：
```
@extends('layouts.app')

@section('content')
  <h2>トップページ</h2>
@endsection
```
これで@yield('content') の場所に、@section('content') の中身が入る。

### ざっくりまとめ
- @include：パーツ読み込み
- @yield：子ビューの内容を入れる穴
- @section：yieldに入れる中身

## 参考サイト
https://engineering.mobalab.net/2019/03/08/laravel%E3%81%AE%E3%83%86%E3%83%B3%E3%83%97%E3%83%AC%E3%83%BC%E3%83%88%E3%81%A7%E3%81%AEinclude-yield-section%E3%81%AE%E9%81%95%E3%81%84/<br>
https://qiita.com/makies/items/2ab24188e7f8482bfddc
