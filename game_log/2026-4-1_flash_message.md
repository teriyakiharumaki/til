# フラッシュメッセージ

## ✅ 行ったこと

- フラッシュメッセージの実装について

## ①コントローラーにメッセージをセッションに保存する
game_log/app/Http/Controllers/GameController.php内の3箇所のredirectを修正

`public function destroy(Game $game)`
```
return redirect()
    ->route('games.index')
    ->with('success', '記録を削除しました！');
```

`public function update(Request $request, Game $game)`
```
return redirect()
    ->route('games.index')
    ->with('success', '記録を更新しました！');
```

`public function store(Request $request)`
```
return redirect()
    ->route('games.index')
    ->with('success', '登録しました！');
```

セッションに保存している部分は、`with`

`with()`は、**次のリクエストで使うために、セッションに一時的なデータを入れるメソッド**

### ②Bladeにフラッシュメッセージを表示
game_log/resources/views/layouts/app.blade.phpの`<main>`内に追記

```
@include('partials.header')

<main>
  @if(session('success'))
    <div style="
        background:#e6fffa;
        border:1px solid #38b2ac;
        padding:10px;
        margin-bottom:15px;
        border-radius:5px;
    ">
        {{ session('success') }}
    </div>

```

- `session('success')`

  セッションに保存された success の値を取り出す

- `@if(session('success'))`

  値があるときだけ表示する

## 参考サイト

- 全ての流れを解説（https://zenn.dev/hiromichinomata/books/f6dfab1ce92272/viewer/e6e93b）

- 別の解説サイト（https://qiita.com/ko-suke2020/items/dff3bde454e7dd1760bf）