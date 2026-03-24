# プレイ時間の表示、保存方法の変更

## ✅ 行ったこと

- プレイ時間を分で登録するのは不便なため、時間と分の入力で登録できるようにした

## ①フォームの変更
game_log/resources/views/games/create.blade.php
```
<div>
  <label>プレイ時間（分）</label><br>
  <input type="number" name="play_time_minutes" min="0" value="{{ old('play_time_minutes') }}">
</div>
```

これを以下のように変更

```
<div>
  <label>プレイ時間</label><br>

  <input type="number" name="play_time_hours" min="0" 
         value="{{ old('play_time_hours') }}" style="width:80px;">
  時間

  <input type="number" name="play_time_minutes_part" min="0" max="59"
         value="{{ old('play_time_minutes_part') }}" style="width:80px;">
  分
</div>
```
- hours (時間の部分)
- minutes_part（分の部分）

時間の部分と、分の部分を分けて送信する

## ②Controllerで変換する
game_log/app/Http/Controllers/GameController.php

```
public function store(Request $request)
```
この部分に以下を追記
```
$hours = (int) $request->input('play_time_hours', 0);
$minutes = (int) $request->input('play_time_minutes_part', 0);

$validated['play_time_minutes'] = $hours * 60 + $minutes;
```

**時間＋分の入力を「分」に変換してDB用データにする処理**

### `$hours = (int) $request->input('play_time_hours', 0);`

- play_time_hours（時間）を取得
- なければ 0
- (int) で整数に変換

(int)について（https://fikaweb.jp/dev/php-convert-int/）

### `$minutes = (int) $request->input('play_time_minutes_part', 0);`

上と同様の処理

### `$validated['play_time_minutes'] = $hours * 60 + $minutes;`

**時間を分に変換して合計している**

なぜ $validated に入れてるのか

```
Game::create($validated);
```
で保存するため

## 今更だけど`Game::create($validated);`でなんで保存できる？

### Gameとは？
```
class Game extends Model
```
**Model = DBのテーブルと対応しているクラス**

| Model | テーブル |
| ---- | ---- |
| Game | games |

***

### create() とは？
```
Game::create($validated);
```

**「新しいレコードを作ってDBに保存する」メソッド**

内部的にやってること
```
INSERT INTO games (...) VALUES (...)
```

***

### `$validated` の役割
```
$validated = [
  'title' => 'ゼルダ',
  'rating' => 5,
  ...
];
```
**ただの保存するデータのセット**

これをそのままDBに入れる

## `Game`のモデル？の部分について

**Model = データベースのテーブルを操作するためのクラス**

今のイメージ

- テーブル → games
- モデル → Game

Game（モデル）
   ↓
games（テーブル）

**1つのModel = 1つのテーブル**

(https://udemy.benesse.co.jp/development/system/laravel-model.html?utm_source=chatgpt.com)

***

### Modelは「DB操作の窓口」
```
class Game extends Model
```
これは、Laravelの特別なクラス（Model）を継承してる

できること

- DBからデータ取得
- DBにデータ保存
- 更新・削除
- 条件検索

***

### Eloquent（ORM）
Laravelは Eloquent（ORM） を使っている

**ORM =「オブジェクト（PHP）とDB（SQL）をつなぐ仕組み」**

(https://zenn.dev/b_tm/articles/7fc9dec1ab3e2a?utm_source=chatgpt.com)

### 内部の構造
```
Game::create()
  ↓
Eloquent
  ↓
SQL生成
  ↓
DBに送る
```