# 一覧にプレイ状況の各ステータスの合計を表示

## ✅ 行ったこと

- 一覧にプレイ状況の各ステータスの合計を表示させた

## ①コントローラーで件数を集計
game_log/app/Http/Controllers/GameController.php

```
$statusCounts = Game::selectRaw('status, COUNT(*) as count')
->groupBy('status')
->pluck('count', 'status');
return view('games.index', compact('games', 'totalMinutes', 'statusCounts'));
```

### `$statusCounts = Game::selectRaw('status, COUNT(*) as count')`

statusごとの件数を数えるための準備をしている部分

#### `selectRaw(...)`


#### `'status, COUNT(*) as count'`

- status を取得する
- COUNT(*) で件数を数える
- その件数に count という名前を付ける

SQLでは、
```
SELECT status, COUNT(*) as count
FROM games
```

#### `COUNT(*)`

**レコード数を数える**

#### `count`

これは別名をつけており、件数の列を `count` という名前で扱えるようにしている

### `->groupBy('status')`

**statusごとにグループ分けして集計する**

これがないとどうなる？`COUNT(*) は全体件数しか返せない。`

でも集計したいのは

- 未プレイは何件？
- プレイ中は何件？
- クリア済みは何件？

だから status ごとに分ける必要がある

SQLでは、
```
SELECT status, COUNT(*) as count
FROM games
GROUP BY status;
```
となる

### `->pluck('count', 'status');`

**取得した結果を「status => count」の形に変換している**

普通はこんな感じのコレクションになる
```
[
  ['status' => 'unplayed', 'count' => 2],
  ['status' => 'playing', 'count' => 1],
  ['status' => 'cleared', 'count' => 3],
]
```

pluck('count', 'status') をすると、

```
[
  'unplayed' => 2,
  'playing' => 1,
  'cleared' => 3,
]
```

これを行うことで、Bladeで次のように書けるようになる

```
{{ $statusCounts['unplayed'] ?? 0 }}
```

引数の順番は

```
pluck('count', 'status')
```

- 値 → count
- キー → status

**status をキー、count を値にした配列っぽい形にする**
