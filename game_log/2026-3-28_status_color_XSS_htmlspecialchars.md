# statusのデザインを変更

## ✅ 行ったこと

- 一覧画面にあるstatusのデザインを変更した

## 元のコード
game_log/resources/views/games/index.blade.php
```
- {{ ['unplayed'=>'未プレイ','playing'=>'プレイ中','cleared'=>'クリア済み'][$game->status] ?? $game->status }}
```

## 変更後
```
@if($game->status === 'unplayed')
  <span style="color: gray;">未プレイ</span>
@elseif($game->status === 'playing')
  <span style="color: orange;">プレイ中</span>
@elseif($game->status === 'cleared')
  <span style="color: green;">クリア済み</span>
@else
  <span>{{ $game->status }}</span>
@endif
```

### ① `@if($game->status === 'unplayed')`
if文
```
$game->status === 'unplayed'
```
「ステータスが未プレイか？」

### ② `<span style="color: gray;">未プレイ</span>`
- `<span>` → インライン要素（文字を囲う）
- `style="color: gray;"` → 文字色を灰色に

未プレイはグレー表示になる

### ③ `@elseif($game->status === 'playing')`

「プレイ中か？」

### ④ `color: orange`

プレイ中はオレンジ

### ⑤ `@elseif($game->status === 'cleared')`

「クリア済みか？」

### ⑥ `color: green`

クリア済みは緑

### ⑦ `@else`

想定外の値が来たとき
```
<span>{{ $game->status }}</span>
```
そのまま表示

## 今更だけど、`{{}}`をなんで使うのか
XSS対策のエスケープ対策を行うため、htmlspecialchars() 関数が使われる

Laravelでは htmlspecialchars関数の代わりに、変数を波カッコで囲むだけ

(https://biz.addisteria.com/laravel_escape/)<br>
詳しい解説がある
