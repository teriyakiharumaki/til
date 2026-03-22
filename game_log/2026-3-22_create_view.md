# 作成画面のviewについて

## ✅ 行ったこと

- 作成画面のviewについてまとめた

## createを作成
```
@extends('layouts.app')

@section('content')
  <h2>ゲーム登録</h2>

  @if ($errors->any())
    <ul>
      @foreach ($errors->all() as $error)
        <li>{{ $error }}</li>
      @endforeach
    </ul>
  @endif

  <form method="POST" action="{{ route('games.store') }}">
    @csrf

    <div>
      <label>ゲーム名 *</label><br>
      <input type="text" name="title" value="{{ old('title') }}" required>
    </div>

    <div>
      <label>プラットフォーム</label><br>
      <input type="text" name="platform" value="{{ old('platform') }}">
    </div>

    <div>
      <label>ステータス *</label><br>
      <select name="status" required>
        <option value="unplayed" @selected(old('status','unplayed')==='unplayed')>未プレイ</option>
        <option value="playing"  @selected(old('status')==='playing')>プレイ中</option>
        <option value="cleared"  @selected(old('status')==='cleared')>クリア済み</option>
      </select>
    </div>

    <div>
      <label>評価（1〜5）</label><br>
      <input type="number" name="rating" min="1" max="5" value="{{ old('rating') }}">
    </div>

    <div>
      <label>感想</label><br>
      <textarea name="review">{{ old('review') }}</textarea>
    </div>

    <div>
      <label>プレイ時間（分）</label><br>
      <input type="number" name="play_time_minutes" min="0" value="{{ old('play_time_minutes') }}">
    </div>

    <button type="submit">登録</button>
  </form>

  <p><a href="{{ route('games.index') }}">← 一覧へ戻る</a></p>
@endsection
```

### ①`@extends('layouts.app')`
共通レイアウト（app.blade.php）を使う

`layouts/app.blade.php`の中身がはめ込まれる

### ②`@section('content')`
layoutの「content部分」に差し込む

`@yield('content')`にこの中身が入る

### ③タイトル表示
```
<h2>ゲーム登録</h2>
```
そのまま

### ④エラーメッセージ表示
```
@if ($errors->any())
```

**バリデーションエラーがあるか？**

```
$errors->any()
```

**一つでもエラーがあればtrue**

```
@foreach ($errors->all() as $error)
```

**エラー一覧を全部取り出す**

```
<li>{{ $error }}</li>
```

**エラーが表示**

### ⑤formタグ
```
<form method="POST" action="{{ route('games.store') }}">
```

**`method="POST"`**：データを送る（登録用）

**`route('games.store')`**： /games に送信

→ Controllerの store() が呼ばれる

### ⑥ @csrf
```
@csrf
```

**セキュリティ用トークン**

これがないと、419エラーになる

csrfについて（https://www.ipa.go.jp/security/vuln/websecurity/csrf.html）

CSRF（Cross Site Request Forgery）<br>
サイトを跨いだなりすまし攻撃

「このフォームは本当にこのサイトから送られたか？」を確認することにより、対策してるシステム

### ⑦ input（ゲーム名）
```
<input type="text" name="title" value="{{ old('title') }}" required>
```

**`name="title"`**：これがControllerに送られるキー

**`value="{{ old('title') }}"`**：入力失敗したときに値を復元

### ⑧ platform
```
<input type="text" name="platform" value="{{ old('platform') }}">
```

同じ仕組み

### ⑨ status（select）
```
<select name="status" required>
```

**`option`**

```
<option value="playing" @selected(old('status')==='playing')>
```

**`@selected の意味`**：条件がtrueなら selected を付ける

例
old('status') === 'playing'

→ 前回選んだ値を復元

### ⑩ rating
```
<input type="number" name="rating" min="1" max="5">
```

**数値入力**

### ⑪ review
```
<textarea name="review">{{ old('review') }}</textarea>
```

**textareaは valueじゃなく中に書く**

### ⑫ play_time
```
<input type="number" name="play_time_minutes" min="0">
```

**分単位**

### ⑬ 送信ボタン
```
<button type="submit">登録</button>
```

**押すとPOST送信**

### ⑭ 戻るリンク
```
<a href="{{ route('games.index') }}">
```

**一覧へ戻る**