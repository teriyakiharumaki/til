# 一覧画面の表示順の変更

## ✅ 行ったこと

- 一覧画面の表示順を新規順に変更

## 元のコード
```
public function index()
    {
        $games = Game::orderByDesc('id')->get();
        return view('games.index', compact('games'));
    }
```

```
$games = Game::orderByDesc('id')->get();
```

**id を降順（DESC）で並べて取得する**

### `orderByDesc('id')`

SQL的には
```
ORDER BY id DESC
```

idが大きいものから上に来る

### `->get()`

DBから全部取得

## 変更後のコード
```
$games = Game::latest()->get();
```

### `latest()`

```
orderBy('created_at', 'desc')
```

のショートカット

## 違い

並び基準が`id`か`created_at`かの違い

`created_at` を使うのが自然なので、使った方がいい