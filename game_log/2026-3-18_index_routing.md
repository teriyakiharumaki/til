# 一覧画面実装の際のルーティングについて

## ✅ 行ったこと

- 一覧画面実装の際のルーティングについてまとめた

## `game_log/routes/web`に以下のルートを追加

```

<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\GameController;

/*
|--------------------------------------------------------------------------
@@ -16,3 +17,7 @@
Route::get('/', function () {
    return view('home');
});

Route::get('/games', [GameController::class, 'index'])->name('games.index');                  以下の三つを追加
Route::get('/games/create', [GameController::class, 'create'])->name('games.create');
Route::post('/games', [GameController::class, 'store'])->name('games.store');
```

## 各部分の意味

### `use Illuminate\Support\Facades\Route;`

**Route クラスを使えるようにしている**

```
Route::get(...)
```

これが使えるのは、この use があるから

### `use App\Http\Controllers\GameController;`

上に同じ

```
[GameController::class, 'index']
```
useがあるからこのように書ける、ない場合

```
[App\Http\Controllers\GameController::class, 'index']
```
となってしまう

### `Route::get('/', fn () => redirect()->route('home'));`

- `Route::get`

  「GETリクエストのとき」

- `/`

  URLのパス（`http://localhost:8000/`）

- `function () {return view('home');`

  viewのhomeを返す

### `Route::get('/games', [GameController::class, 'index'])->name('games.index');`

- `/games`

  URL：`http://localhost:8000/games`

- `[GameController::class, 'index']`

  **GameController の index メソッドを実行する**

  `public function index()`が呼ばれる

- `->name('games.index')`

  **ルートの名前をつけている**

  `route('games.index')`と書ける


- `Route::get('/games/create', [GameController::class, 'create'])->name('games.create');`, `Route::post('/games', [GameController::class, 'store'])->name('games.store');`

  上と同じ流れ

## まとめ

```
GET  /           → home表示
GET  /games      → index() 一覧表示
GET  /games/create → create() フォーム表示
POST /games      → store() DB保存
```
