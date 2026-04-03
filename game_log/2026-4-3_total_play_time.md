# 一覧に総プレイ時間の実装

## ✅ 行ったこと

- 一覧に総プレイ時間の実装

## ①コントローラーに変数を渡す

game_log/app/Http/Controllers/GameController.php
```
public function index()
{
    $games = Game::latest()->get();
    $totalMinutes = Game::sum('play_time_minutes');
    return view('games.index', compact('games', 'totalMinutes'));
}
```

### `$totalMinutes = Game::sum('play_time_minutes');`

ここが総プレイ時間の本体となる部分

#### `sum('play_time_minutes')`

**games テーブルの play_time_minutes カラムを全部足し算する**

LaravelのEloquentが、**SQLの SUM() を使って合計値を取る**

Eloquentは、**SQLをPHPっぽく書けるようにしたもの**

### `return view('games.index', compact('games', 'totalMinutes'));`

**Viewにデータを渡して画面を表示する部分**

#### `compact('games', 'totalMinutes')`

```
[
  'games' => $games,
  'totalMinutes' => $totalMinutes,
]
```
と同じ意味、つまりView側で
```
$games
$totalMinutes
```
が使えるようになる

## ②indexのviewに表示

game_log/resources/views/games/index.blade.php
```
@php
  $h = intdiv($totalMinutes, 60);
  $m = $totalMinutes % 60;
@endphp

<div style="margin-bottom:15px; padding:10px; background:#f0f9ff;">
  🎮 総プレイ時間：
  @if($h > 0) {{ $h }}時間 @endif
  @if($m > 0) {{ $m }}分 @endif
</div>
```

### `@php`部分

```
@php
  $h = intdiv($totalMinutes, 60);
  $m = $totalMinutes % 60;
@endphp
```

計算する部分

#### `$h = intdiv($totalMinutes, 60);`

時間（hour）を出してる部分

`intdiv()`について（）

#### `$m = $totalMinutes % 60;`

60で割った余り（分）を出してる部分

### `表示する部分`

```
<div>
  🎮 総プレイ時間：
  @if($h > 0) {{ $h }}時間 @endif
  @if($m > 0) {{ $m }}分 @endif
</div>
```

#### `@if($h > 0)`, `@if($m > 0)`

時間、分は0の時は表示しない
