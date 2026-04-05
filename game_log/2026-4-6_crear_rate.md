# 一覧にクリア率を表示

## ✅ 行ったこと

- 一覧にクリア率を表示させた

## ①コントローラーでクリア率の計算
game_log/app/Http/Controllers/GameController.php
```
$total = Game::count();
$cleared = Game::where('status','cleared')->count();

$clearRate = $total ? round($cleared / $total * 100) : 0;

return view('games.index', compact(
    'games',
    'totalMinutes',
    'statusCounts',
    'clearRate'
));
```

### `$total = Game::count();`

総ゲーム数を数えている（games テーブルの件数を数えている）

### `$cleared = Game::where('status','cleared')->count();`

クリア済みの数を数えている（status = cleared のものだけ数える）

上と同じcountについて（https://gyas.co.jp/knowledge_blog/laravel-eloquent-count/）

### `$clearRate = $total ? round($cleared / $total * 100) : 0;`

クリア率の計算をしている

- `$cleared / $total * 100`

  この部分で、割合をパーセントに変換

- `round(...)`

  小数を四捨五入

  (https://www.sejuku.net/blog/48961#index_id0)

- `$total ? A : B`

  totalが0じゃなければA、0ならB<br>
  (ゲームが0件だと、0/0となりエラーになる)

  これは**三項演算子**<br>
  条件式を一文で書ける

  (https://qiita.com/sho91/items/c88c3418f777385f962c)
  (https://qiita.com/shonansurvivors/items/1e3194cf3eb2ea089039)

## ②一覧に表示
game_log/resources/views/games/index.blade.php
```
<div style="margin-bottom:10px;">
  🏆 クリア率：{{ $clearRate }}%
</div>
```
