# プレイ状況絞り込み

## ✅ 行ったこと

- プレイ状況によって絞り込む機能の追加を行なった

## ①URLでステータスの場合分けを判別できるように修正
docker/nginx/default.conf

```
  location / {
    root /var/www/game_log/public;
    index index.php;
    try_files $uri $uri/ /index.php?$query_string;        #ここが大事
  }

  location ~ .php$ {
```

大事な部分は、
```
try_files $uri $uri/ /index.php?$query_string;  
```
であり、**ファイルがあればそれを返す。なければLaravelに処理を渡す**

### `try_files`

```
try_files A B C;
```

意味：**Aを探す → なければB → なければC**

### `$uri`

```
/css/app.css
/images/logo.png
```
つまり、実際のファイルを探す

### `$uri/`

これは、ディレクトリとして存在するか確認

### `/index.php?$query_string`

- `/index.php`

  全部のリクエストをLaravelに渡す

  Laravelは、`public/index.php`から始まる

- `?$query_string`

  **URLの ? 以降をそのまま引き継ぐ**

  例：
  ```
  /games?status=cleared

  ↓

  /index.php?status=cleared
  ```

### 実際の流れ

例：/games?status=cleared

1. `nginx`
  `/games` というファイルある？
  → ない

2. 次
  `/games/` というディレクトリある？
  → ない

3. 最後
  `/index.php?status=cleared` に送る

4. Laravel
  `request('status') → "cleared"`
  ここでやっと取得

### クエリなしでindexに飛ぶ時
```
http://localhost:8000/games
```
クエリなし（?以降なし）

`/games`の場合の流れ

1. `nginx`
  `/games` というファイルある？
  → ない

2. 次
  `/games/` というディレクトリある？
  → ない

3. 最後
  `/index.php?status=cleared` に送る<br>
  ここで`$query_string = 空`だから、`/index.php`になる

## ②コントローラーで場合分けの処理
game_log/app/Http/Controllers/GameController.php
```
{
    public function index()
    {
        $query = Game::query();

        if (request()->filled('status')) {
            $query->where('status', request('status'));
        }

        $games = $query->latest()->get();
```

### `$query = Game::query();`

```
$query = Game::query();
```

Game はモデルだから、games テーブルを操作するための入り口

query() は何？：**これから検索条件を組み立てますよ**

(https://qiita.com/fujita-goq/items/2279bb947ec4e7b103b2)<br>
(https://zenn.dev/nshiro/articles/98d3826151af81)<br>
(https://dexall.co.jp/articles/?p=2706)

### `if (request()->filled('status'))`

ここは、**URLに status が入っていたら**という意味

- `request()`

  今のリクエスト情報を取るためのもの

  たとえばURLが
  ```
  /games?status=cleared
  ```
  なら、`request('status')`で `"cleared"` が取れる

- `filled('status')`

  その値が空じゃないかを確認してる

  つまり
  ```
  request()->filled('status')
  ```
  は、status がちゃんと送られていて、中身も空じゃないなら true

  例：URLが以下なら
  ```
  /games?status=playing

  → true
  ```

  URLが以下なら
  ```
  /games

  → false
  ```

###  `$query->where('status', request('status'));`

これは、**status カラムが、送られてきた status と一致するものだけにする**という意味

例：URLが
```
/games?status=cleared
```
なら、
```
$query->where('status', 'cleared');
```
と同じになる

つまりSQLっぽく言うと：
```
WHERE status = 'cleared'
```

### `$games = $query->latest()->get();`

ここで初めて 実際にデータを取っている

- latest()

  **新しい順に並べる**という意味

  created_at DESC と同じ

- get()

  **検索結果を実際に取り出す**という意味

  ここまではずっと「条件を作ってるだけ」だったけど、
  get() がついた瞬間に、DBに問い合わせて結果が返ってくる

参考サイト（https://readouble.com/laravel/8.x/ja/queries.html）
