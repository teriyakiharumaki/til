# 編集機能の実装

## ✅ 行ったこと

- 投稿の編集機能を実装した

## ①ルーティング
game_log/routes/web.php
```
Route::get('/games/{game}/edit', [GameController::class, 'edit'])->name('games.edit');
Route::put('/games/{game}', [GameController::class, 'update'])->name('games.update');
```
この二行を実装

***

```
Route::get('/games/{game}/edit', [GameController::class, 'edit'])->name('games.edit');
```

### `Route::get(...)`

**GETリクエストのときに動くルート**

GETは、ブラウザで普通にページを開くときのリクエスト

### `'/games/{game}/edit'`

URLパス

`/edit`で編集画面であることを表す

{game}は、2026-3-26_delete.mdを参照

### `[GameController::class, 'edit']`

**GameController の edit メソッドを呼ぶ**

### `->name('games.edit')`
ルートに名前をつけている、Bradeで
```
route('games.edit', $game)
```
と書けるようになる

***

```
Route::put('/games/{game}', [GameController::class, 'update'])->name('games.update');
```

### `Route::put(...)`

**PUTリクエストのときに動くルート**

PUT は、「既存データを更新する」ときによく使うHTTPメソッド

### その他の要素はeditと同じ

## ②コントローラーにeditとupdateを追記
game_log/app/Http/Controllers/GameController.php
```
public function edit(Game $game)
{
    return view('games.edit', compact('game'));
}

public function update(Request $request, Game $game)
{
    $validated = $request->validate([
        'title' => ['required', 'string', 'max:255'],
        'platform' => ['nullable', 'string', 'max:255'],
        'status' => ['required', 'in:unplayed,playing,cleared'],
        'rating' => ['nullable', 'integer', 'min:1', 'max:5'],
        'review' => ['nullable', 'string'],
        'play_time_hours' => ['nullable', 'integer', 'min:0'],
         'play_time_minutes_part' => ['nullable', 'integer', 'min:0', 'max:59'],
    ]);

    $hours = (int) ($validated['play_time_hours'] ?? 0);
    $minutes = (int) ($validated['play_time_minutes_part'] ?? 0);
    $validated['play_time_minutes'] = $hours * 60 + $minutes;

    unset($validated['play_time_hours'], $validated['play_time_minutes_part']);

    $game->update($validated);

    return redirect()->route('games.index');
}
```

***

```
public function edit(Game $game)
{
    return view('games.edit', compact('game'));
}
```

### `public function edit(...)`

edit という名前のメソッド

ルーティングの
```
Route::get('/games/{game}/edit', ...)
```
と対応してる

つまり、**/games/1/edit にアクセスしたとき、この edit() が動く**

### `Game $game`

ここは Route Model Binding によって、

/games/1/edit

なら

Laravel が自動でDBから1番のGameを探してくれて、その結果を $game に渡している。

つまり $game の中には、

```
title
platform
rating
review
play_time_minutes
```
みたいな、1件分のゲームデータが入ってる。

Route Model Bindingについて（https://zenn.dev/taroosg/articles/20240925175812-85e66e02a13362）

### `return view('games.edit', compact('game'));`

**games/edit.blade.php を表示する**<br>
**そのとき、$game という変数をViewに渡す**

という意味

### `compact('game')` って何？

これは PHP の関数で、以下のような意味
```
['game' => $game]
```
つまり、Controllerの中の $game を、Viewでも $game として使えるようにしてる

***

```
public function update(Request $request, Game $game)
{
    $validated = $request->validate([
        'title' => ['required', 'string', 'max:255'],
        'platform' => ['nullable', 'string', 'max:255'],
        'status' => ['required', 'in:unplayed,playing,cleared'],
        'rating' => ['nullable', 'integer', 'min:1', 'max:5'],
        'review' => ['nullable', 'string'],
        'play_time_hours' => ['nullable', 'integer', 'min:0'],
         'play_time_minutes_part' => ['nullable', 'integer', 'min:0', 'max:59'],
    ]);

    $hours = (int) ($validated['play_time_hours'] ?? 0);
    $minutes = (int) ($validated['play_time_minutes_part'] ?? 0);
    $validated['play_time_minutes'] = $hours * 60 + $minutes;

    unset($validated['play_time_hours'], $validated['play_time_minutes_part']);

    $game->update($validated);

    return redirect()->route('games.index');
}
```

### `update(Request $request, Game $game)`

フォームから送られてきたデータを受け取って、対象ゲームを更新する処理

### `Request $request`

**フォームの入力値が入ってるもの**

フォームで入力した
```
title
platform
status
rating
review
play_time_hours
play_time_minutes_part
```
がここに入っている

### `Game $game`

これも edit() と同じで、URLの {game} に対応したゲームデータ
```
PUT /games/1
```
なら、1番のゲームレコードが `$game` に入る

### バリデーション部分
```
$validated = $request->validate([
    'title' => ['required', 'string', 'max:255'],
    'platform' => ['nullable', 'string', 'max:255'],
    'status' => ['required', 'in:unplayed,playing,cleared'],
    'rating' => ['nullable', 'integer', 'min:1', 'max:5'],
    'review' => ['nullable', 'string'],
    'play_time_hours' => ['nullable', 'integer', 'min:0'],
    'play_time_minutes_part' => ['nullable', 'integer', 'min:0', 'max:59'],
]);
```

'nullable'：空でもOK

'string'：文字列であること

'integer'：整数であること

'max:255'：255文字以内

'min:1'：1以上

'max:5'：5以下

'in:unplayed,playing,cleared'：この3つのどれかであること

### プレイ時間の変換
```
$hours = (int) ($validated['play_time_hours'] ?? 0);
$minutes = (int) ($validated['play_time_minutes_part'] ?? 0);
$validated['play_time_minutes'] = $hours * 60 + $minutes;
```

フォームでは○時間○分に分けて入力していたが、DBには `play_time_minutes` 1つだけで保存するのでここで変換している

### `$validated['play_time_hours'] ?? 0`

**play_time_hours があればその値、なければ 0**

?? は「なければこれ」

### `(int)`

整数に変換("2" みたいな文字列でも 2 にできる)

### `$hours * 60 + $minutes`

分数を計算し、代入

### `unset($validated['play_time_hours'], $validated['play_time_minutes_part']);`

**もう不要になった入力用の値を消す**

DBにあるカラムは

**play_time_minutes**

だけで、

**play_time_hours**<br>
**play_time_minutes_part**

というカラムは存在しない

もしそのまま update() に渡すと、不要な項目まで更新しようとしてしまうので、

**入力用の仮データは削除し、代わりに play_time_minutes だけ残す**

### `$game->update($validated);`

**$game のレコードを、$validated の内容で更新する**

### リダイレクト

```
return redirect()->route('games.index');
```

**更新が終わったら、ゲーム一覧画面に戻る**
